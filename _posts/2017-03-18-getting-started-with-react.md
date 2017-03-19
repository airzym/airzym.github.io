---
layout: post
title:  "React, first steps (Webpack and Babel)"
date:   2017-03-18 14:12:00
author: Aaron Bamford
categories: Javascript
tags:	[react, es6, webpack, babel]
---

Getting started with React can be a big hurdle. Before you can get started creating your react app, youll need an environment that compiles ES6 into Javascript that can run in your browser. For this tutorial, I'll be using [Babel](http://babeljs.io) and [Webpack](https://webpack.js.org) to do this.

### Prerequisites
For the tutorial, you'll need [node](https://nodejs.org/en/) installed. (I'm using version 6.10.0)

## Initialize your Node Project
In your project directory run

```
npm init
```

This will initialize our  `package.json` with a few default properties. Don't worry about most the settings it asks for to begin with, just give it a name of whatever you'd like to call your project.

This should start us off with a `package.json` looking something like this:
{% highlight json %}
{
  "name": "react-redux-starter",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Aaron Bamford",
  "license": "ISC"
}
{% endhighlight %}

Theres a few node packages we're going to need to install. I'll do my best to explain each package and why we need it.

- [`babel-core`](https://www.npmjs.com/package/babel-core): This is the core package used to transpile your ES6 into Javascript for your browser.
- [`babel-loader`](https://www.npmjs.com/package/babel-loader): This allows babel to be used with webpack.
- [`babel-preset-es2015`](https://www.npmjs.com/package/babel-preset-es2015): This [preset](https://babeljs.io/docs/plugins/#presets) is what tell babel how to transpile most of the ES6 code we'll be writing.
- [`babel-preset-react`](https://www.npmjs.com/package/babel-preset-react): This preset tell babel how to transpile the React code.
- [`webpack`](https://www.npmjs.com/package/webpack): This allows us to bundle our Javascript we want to use in the browser.

As these packages are all development devepdancies only we'll install them doing the following:

```
npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react webpack
```

Our `package.json` should now have the following packages:
{% highlight json %}
{
  ...
  "devDependencies": {
    "babel-core": "^6.24.0",
    "babel-loader": "^6.4.1",
    "babel-preset-es2015": "^6.24.0",
    "babel-preset-react": "^6.23.0",
    "webpack": "^2.2.1"
  }
}
{% endhighlight %}

## Babel
Lets first configure babel so it knows how to transpile our code how we need it to. We'll create a `.babelrc` file for our babel settings.

This file just contains some json with some settings which tell babel how to behave. For now all we need to do is specify the two presets we want to use.

{% highlight json %}
{
  // This is why we installed the two preset babel packages
  "presets": ["es2015", "react"]
}
{% endhighlight %}

## Webpack
Now we need to set up webpack to use babel and bundle the javascript for us to use in our site. We'll create a `webpack.confg.js` file. This is where we'll export our webpack configuration.

{% highlight javascript %}
// First we'll need webpack.
var webpack = require('webpack');

// These are the settings for webpack to tell it what to do
module.exports = {
  // This is the entry point of your application (dont worry, we'll create this next!)
  entry: './app/index.js',
  output: {
    path: require('path').resolve('./dist'),
    filename: 'bundle.js',
    publicPath: '/'
  },
  module: {
    rules: [{
      // this rule will apply to all .js files
      test: /\.js$/,
      // we dont want anything in /node_modules/ to be included
      exclude: /node_modules/,
      // and we want to run these files through babel (this is where that babel-loader package comes in)
      loader: "babel-loader"
    }]
  }
};
{% endhighlight %}

## React
At this point we can start thinking about writing some React code. For that we'll need to install the [react](https://www.npmjs.com/package/react) and [react-dom](https://www.npmjs.com/package/react-dom) node packages.

```
npm i --save react react-dom
```
Our `packages.json` should now have the following:
{% highlight json %}
{
  ...
  "dependencies": {
    "react": "^15.4.2",
    "react-dom": "^15.4.2"
  }
}
{% endhighlight %}

Because I want the entry point of my application to be in `app/index.js` we'll create that first.

{% highlight javascript %}
import React from 'react';
import { render } from 'react-dom';

render(
  <h1>Hello World!</h1>,
  document.getElementById('app')
);
{% endhighlight %}

Now we just need a html file to use our Javascript. Because I want the `bundle.js` to be output to a `dist` directory. We'll create an `index.html` there.

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>React Redux Starter</title>
</head>
<body>
  <!-- This is where our react components will be rendered -->
  <div id="app"></div>
  <!-- We'll need reference our bundled javascript -->
  <script src="./bundle.js"></script>
</body>
</html>
{% endhighlight %}

## Last Steps
Finally, in order for us to compile the javascript, we need to install the webpack package globally:
```
npm i webpack -g
```

We can then just run:
```
webpack
```
This should create `bundle.js` in `/dist`. Go ahead and open `index.html` in your browser and you should see "Hello World!".

<img src="/assets/2017-03-18-getting-started-with-react/hello-world.png" title="Hello World!">

Thats all you'll need to get started to develop your ReactJS app.


Find the source code for this over at [github here](https://github.com/airzy91/react-redux-starter/tree/first-steps).
