---
layout: post
title: Custom authentication flow with Sorcery gem RSpec and Rails 7 part 1
subtitle: Creating the registration process and auto login for user
categories: code
author: adrian
date: 2022-01-10 02:20:00 -0700
---

We're going to install Sorcery, a low-level authentication gem to help you create a custom authentication flow for your app

<sl-responsive-media aspect-ratio="16:9">
<iframe width="560" height="315" src="https://www.youtube.com/embed/bhs1FVkTBXw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</sl-responsive-media>


I'm picking up where I left off in a post where I install RSpec onto a fresh Rails 7 app. If you want to start there so we are on the same page please feel free: [Install RSpec on Rails 7](https://dev.to/adrianvalenz/setup-rspec-on-a-fresh-rails-7-project-5gp)

```
git checkout -b sorcery-sign-up-feature
bundle add sorcery
bundle install
rails g sorcery:install
rails db:migrate
```

Before we write any code let's just write a simple spec to serve as our guide so we know what to focus on energy on.

## Spec

```
# spec/system/sign_up_spec.rb

require 'rails_helper'

describe "User signs up", type: :system do
  let(:email) { Faker::Internet.email }
  let(:password) { Faker::Internet.password(min_length: 8) }

  before do
    visit root_path
  end

  scenario "register user account" do
    click_on "Register"
    fill_in "Email", with: email
    fill_in "Password", with: password
    fill_in "Password confirmation", with: password
    click_button "Sign up"

    expect(page).to have_content("Dashboard")
    expect(page).to have_content(email)
    expect(page).to have_no_content("Register")
  end
end
```

I'm just going to give you all the code you need to pass that test!

## Routes

```
# config/routes.rb

Rails.application.routes.draw do
  get 'dashboard', to: 'dashboard#show'
  get 'register', to: 'user_registration#new'
  resources :user_registrations, only: :create
  root 'home#show'
end
```

## Controllers

```
# app/controllers/home_controller.rb

class HomeController < ApplicationController
  def show
  end
end
```

```
# app/controllers/dashboard_controller.rb

class DashboardController < ApplicationController
  def show
  end
end
```

```
# app/controllers/user_registrations_controller.rb

class UserRegistrationsController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    
    if @user.save
      auto_login(@user)
      redirect_to dashboard_path, notice: "You have registered successfully."
    else
      render :new
    end
  end

  private
  def user_params
    params.require(:user).permit(:email, :password, :password_confirmation)
  end
end
```

## Views

```
# app/views/home/show.html.erb

<h1>Homepage</h1>
```

```
# app/views/dashboard/show.html.erb

<h1>Dashboard</h1>
```

```
# app/views/shared/_navbar.html.erb

<% if current_user %>
  <%= current_user.email %>
<% else %>
  <%= link_to "Register", register_path %>
<% end %>
```

```
# app/views/layout/application.html.erb

<%= render "shared/navbar" %>
<div>
  <p id="notice"><%= flash[:notice] %></p>
  <p id="alert"><%= flash[:alert] %></p>
</div>
```

```
# app/views/user_registrations/new.html.erb

<h1>Register an account</h1>

<%= form_with(model: @user, url: user_registration_path, method: :post) do |form| %>
  <div>
    <%= form.label :email %>
    <%= form.text_field :email %>
  </div>

  <div>
    <%= form.label :password %>
    <%= form.password_field :password %>
  </div>

  <div>
    <%= form.label :password_confirmation %>
    <%= form.password_field :password_confirmation %>
  </div>

  <div>
    <%= form.submit "Sign up" %>
  </div>
<% end %>
```

## Models

```
# app/model/user.rb

class User < ApplicationRecord
  authenticates_with_sorcery!

  validates :password, length: { minimum: 3 }, if: -> { new_record? || changes[:crypted_password] }
  validates :password, confirmation: true, if: -> { new_record? || changes[:crypted_password] }
  validates :password_confirmation, presence: true, if: -> { new_record? || changes[:crypted_password] }

  validates :email, uniqueness: true
end
```

Let's finish up
```
git status
git add -A
git commit -m "add sorcery sign up and auto login flow"
git checkout main
git merge sorcery-sign-up-feature
git branch -D sorcery-sign-up-feature
git push
```

## Continue reading for my ramblings

By the way I just wanted to point out that sweet `auto_login` helper method Sorcery 
has to automatically log in your user. We slipped it into the create action to create
a nice seamless way to log in a user. Sorcery has the option for user activation via a
confirmation link as well if you wish to incorporate that.

Next time we will create a way to sign out and actually hide pages from users not signed in. 
Right now you can see the dashboard page whether you are logged in or not but in our next 
test we'll focus on that. We'll also add a way to sign out, delete users, sign in, and show form errors. 
The form we made only works for registering accounts, not signing in existing ones. 
If you try using the form again you'll see in the server console that it doesn't let 
you move forward with the action because we set a validates uniqueness on email attributes.

Speaking of attributes, you might have noticed the migration file Sorcery generated only had 3 
attributes: email, salt, and crypted_password. If you are unsure about how we were able to fill in
password and password_confirmation through the form then you might not be alone. This was done with
something called "virtual attributes". The form attributes had values but they were not persisted into 
the database, the password the user wrote was encrypted and that was stored into the database. You never
want to store plain passwords in a database. No-no.

Hope you enjoyed the tutorial! This is the great thing about Sorcery, 
you are in control of your authentication flow and can add or remove as much or little as you like.
