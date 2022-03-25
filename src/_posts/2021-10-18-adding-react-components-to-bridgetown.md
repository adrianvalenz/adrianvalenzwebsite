---
layout: post
title: Time-based greeting with React and Bridgetown
subtitle: Adding a React compononent that renders a greeting based on the user's time of day.
author: adrian
categories: code
---

React is a library that I have been wanting to implement into my Bridgetown site
for quite some time now. Today we're going to configure React into Bridgetown,
and optionally use a component to render a greeting to the users based 
on *their* time of day.

I'll be separating this tutorial into two sections. First section we'll get
React running, second section we will add the fun little greeting script.

*At the time of writing this, I am running Bridgetown version 0.21.4.*

## Part 1: Configure React

### Overview of steps

1. Install the packages via Yarn
2. Update the webpack.config.js
3. Create your component
4. Ensure DOM is loaded before mounting component
5. Add container to layout

### 1. Install the packages via Yarn

`yarn add -D react react-dom`

Run this at the root of your project where the package.json file is located. The
"-D" flag installs them as dev dependencies.


### 2. Update the webpack.config.js

Place this code...
```
var esbuildLoader = config.module.rules.find(rule => rule.use && rule.use.loader == "esbuild-loader")
if (esbuildLoader) {
  esbuildLoader.use.options.loader = "jsx"
}
```
After this...
```
var config = require("./config/webpack.defaults.js")
```
and before this...
```
module.exports = config
```

**Don't** edit the "webpack.defaults.js" file because it can potentially be
overriden when you upgrade Bridgetown to a newer version.

The snippet looks through the module rules in the default config for a loader
called "esbuild-loader" and if it is there then we have the loader use JSX.


### 3. Create your component

Create a folder "components" in your javascript directory. Here you will keep
all of your React compononents that will be imported into index.js.

`./frontend/javascript/components`

Create a file called `HelloWorld.js` and add the following code...

```
# in ./frontend/javascript/components/HelloWorld.js

import React from "react"
import ReactDOM from "react-dom"

export const HelloWorld = () => {
  const hello = "Hello"

  return <p>{hello} World!</p>
}

export const renderInDOM = () => {
  console.log("about to render!")
  ReactDOM.render(<HelloWorld />, document.querySelector("#root"))
}
```

### 4. Ensure DOM is loaded before mounting component

```
# This snippet goes in your `./frontend/javascript/index.js`

import { renderInDOM } from "./components/HelloWorld"

window.addEventListener("DOMContentLoaded", () => {
  renderInDOM()
})
```

Now we are importing the `renderInDOM` function from the component file and
using an event listener to make sure the DOM is loaded so we have somewhere to
render the component.

Note: Thank you [@jaredcwhite](https://twitter.com/jaredcwhite) for helping me
with the webpack.config.js and import DOM snippet to make this happen!

### Add your div to your layout

```
<div id="root"></div>
```

Think about where you want to render your component. It could be in your layouts
or pages, where ever you like. I put mine in my index page. 

There you have it! You should have a "Hello World!" rendering where you 
placed your div tag with the id of "root".

## Part 2: Render a greeting based on time of day

The following is is an example of a custom greeting that changes a string based
on the time of day.

### Overview of steps

1. Create component for greeting users
2. Import into index.js
3. Apply html tag to template

### 1. Create component for greeting users

Let's make another component called `Greeting.js`. Place the following
code inside of it.

```
# In ./frontend/javascript/components/Greeting.js

import React from "react"
import ReactDOM from "react-dom"

export const Greeting = () => {
	var myDate = new Date();
	var hours= myDate.getHours();
	var greet;

	if (hours < 12)
		greet =  "morning";
	else if (hours >= 12 && hours <= 17)
		greet = "afternoon";
	else if (hours >= 17 && hours <= 24)
		greet = "evening";

	return <span>Good {greet}!</span>
}

export const renderGreeting = () => {
	console.log("dynamic Greeting is rendering!")
	ReactDOM.render(<Greeting />, document.querySelector("#greeting"))
}
```

JavaScript is perfect for this feature because it looks for the time of day
based on the browser the user is viewing it on because it is client side
scripting. If we used server side scripting then the time of day based on the
server may not match up with the user viewing it in real time.

So we make a new instance of the date, get the hours, and using if/else
statements we determine which greeting to render based on the hour of the day.
Then we return it via some JSX and Kablam! You have a customize time-based
greeting.

I really like how many of the new JavaScript frameworks encapusalate scripts
into components for specific use-cases. Another awesome library that does this
is [StimulusJS](https://stimulus.hotwired.dev/), but let's finish up and 
put the rest of the pieces together.

### Import into index.js

```
# In ./frontend/javascript/index.js

import { renderGreeting } from "./components/Greeting"

window.addEventListener("DOMContentLoaded", () => {
  renderGreeting()
})
```

Just like last time we import the function from the component and wait/listen
for the DOM to finish loading the content so we have somewhere to render the
greeting.

### Apply html tag to template

```
<div id="greeting"></div>
```

Where we want the greeting to appear depends on where we place the tag, so drop
your tag where ever your heart desires my friend.


