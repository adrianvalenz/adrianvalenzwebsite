---
layout: post
title: Setup RSpec on fresh Rails 7 Project
subtitle: Installing RSpec, FactoryBot, Faker gems and configuring for testing environment
categories: code
author: adrian
---

Let's get right to it...

## Start a fresh project

```
rails new projectname -d postgresql
cd projectname
git add -A
git commit -m "initial commit"

rails db:create
rails s
```

## Install gems

```
git checkout -b rspec
```

In Gemfile add...

```
group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mingw x64_mingw ]
  gem "rspec-rails"
  gem "factory_bot_rails"
  gem "faker"
end
```

Back in terminal/command line...

```
bundle install
rails g rspec:install
```

## Configure gems

Create 'support' directory in 'spec' directory and create two files

```
spec/support/factory_bot.rb
spec/support/chrome.rb
```

And then one file in the spec directory for factories.

```
spec/factories.rb
```

Configure FactoryBot

```
# spec/support/factory_bot.rb

RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
```

Configure driver

```
# spec/support/chrome.rb

RSpec.configure do |config|
  config.before(:each, type: :system) do
    if ENV["SHOW_BROWSER"] == "true"
      driven_by :selenium_chrome
    else
      driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
    end
  end
end
```

Require support files in `rails_helper.rb`

```
# spec/rails_helper.rb

require_relative 'support/factory_bot'
require_relative 'support/chrome'
```

When you generate a User model (or any model) RSpec will generate a factory for you in your `factories.rb` file.
You can modify it to look like this.

```
# spec/factories.rb

FactoryBot.define do
  factory(:user) do
    email { Faker::Internet.email }
    password { Faker::Internet.password }
  end
end
```

## it "should run a test" do...

Run `rspec` command in the terminal and you should see it run and output some green text.

```
-> rspec
No examples found.

Finished in 0.00038 seconds (files took 0.94577 seconds to load)
0 examples, 0 failures
```

## Finish up

```
git add -A
git commit -m "add rspec"
git checkout main
git merge rspec
git branch -D rspec
git push # assuming you have a repo online somewhere
```
