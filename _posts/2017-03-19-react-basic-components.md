---
layout: post
title:  "React, basic components"
date:   2017-03-19 18:35:00
author: Aaron Bamford
categories: Javascript
tags:	[react, es6, babel]
---

In this guide we'll build a simple note component in React. We'll also look at the difference between [Presentation and Container](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.di7pus8vs) components.

### Prerequisites
I'll assume for this you've followed [this tutorial]({% post_url 2017-03-18-getting-started-with-react %}). Or you have a development environment set up that allows your to build your ES6 into javascript for your browser.

## App

We'll start off by creating a component that will be the root component for the rest of our app. Go ahead and create `app.js`.
When creating components the first thing you'll need to `import` is `React`.
{% highlight javascript %}
import React from 'react';
{% endhighlight %}

There are a few ways to define a component. As this is a "presentation" component, I'll create a [stateless functional component](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components). We'll learn how to define a component as a ES6 class later.

{% highlight javascript %}
const App = () => {
   return (
     <h1>
      My React App
    </h1>
  );
};
{% endhighlight %}

And finally we need to export the component

{% highlight javascript %}
export default App;
{% endhighlight %}

We can now use this component in `index.js`. First import the component.
{% highlight javascript %}
import App from './app';
{% endhighlight %}

Then update the arguments passed to the `render` function;
{% highlight javascript %}
render(
  <App/>,
  document.getElementById('app')
);
{% endhighlight %}

If you build your javascript and open up your browser, you should now see your title displayed!

<img src="/assets/2017-03-19-react-basic-components/title.png" title="React Title">

## Note Form

Now lets create our note component, in this component, we need a text input for the note title, and a textarea for the body of the note. So first create a file `noteForm.js`.

First we'll need to import `React` again. However, because this won't be a functional component, we also need to import `Component`. (This is optional, and the same as writing `React.Component`, however I think it cleans up your implementation and reads a lot nicer)
{% highlight javascript %}
import React, { Component } from 'react';
{% endhighlight %}

Next, we'll create our `NoteForm` class
{% highlight javascript %}
class NoteForm extends Component {
  render(){
    return (
      <form>
        <div>
          <label>Title:</label>
          <input type="text"/>
        </div>
        <div>
          <label>Note:</label>
          <textarea/>
        </div>
      </form>
    );
  }
}
{% endhighlight %}

And like we did with the `App` component, we'll need to export it to use elsewhere.
{% highlight javascript %}
export default NoteForm;
{% endhighlight %}

Then we'll update the `App` component to render our new `NoteForm` component.

{% highlight javascript %}
//...
import NoteForm from './noteForm';

const App = () => {
   return (
     <h1>
      My React App
      <NoteForm />
    </h1>
  );
};
//...
{% endhighlight %}

You should be able to build your application at this point and view it in your browser.
<img src="/assets/2017-03-19-react-basic-components/note.png" title="React Note">

## Controlled components.

At the moment our `<textarea/>` and our `<input/>` components manage their own state, and our React component knows nothing about their values. We'll change these to be [controlled components](https://facebook.github.io/react/docs/forms.html), this means our `NoteForm` component will be the single source of truth for their value.

The first thing we'll want to do here is define the state of our component. We do this in the constructor for the class. We can then set the value of our components to the value of the state properties.
{% highlight javascript %}
//...
class NoteForm extends Component{
  constructor(props){
    super(props);
    this.state = {
      title: '',
      value: ''
    };
  }

  render(){
    return (
      <form>
        <div>
          <label>Title:</label>
          <input type="text" value={this.state.title}/>
        </div>
        <div>
          <label>Note:</label>
          <textarea value={this.state.value} />
        </div>
      </form>
    );
  }
}
//...
{% endhighlight %}
You can read about why we need to call `super(props)` [here](https://facebook.github.io/react/docs/react-component.html#constructor).

However, if you run your application now, you'll find when you type in the `<input/>` or the `<textarea/>` nothing happens! This is because we've not told our `NoteForm` component what to do, when we try and update the value. So lets create two functions in our components class definition, and tell our `<input/>` and `<textarea/>` components to call the respective function, when the value is changed.
{% highlight javascript %}
//...
_onTitleUpdate(event){
  this.setState({ title: event.target.value });
}

_onValueUpdate(event){
  this.setState({ value: event.target.value });
}
//...
{% endhighlight %}

{% highlight javascript %}
//...
<input type="text" value={this.state.title} onChange={this._onTitleUpdate.bind(this)}/>
//...
<textarea value={this.state.value} onChange={this._onValueUpdate.bind(this)}/>
//...
{% endhighlight %}

If you run your app now, you should be able to update the values of your text boxes!


### Autobinding
Using ES6 classes means our components don't automatically bind `this` to the instance. This is why have to call `.bind(this)` when passing our methods to `onChange`. Because this adds a lot of boilerplate to our code, it'd be nice to get our components to [autobind](https://facebook.github.io/react/docs/react-without-es6.html#autobinding). To do this we need to install the [tansform-class-properties](https://babeljs.io/docs/plugins/transform-class-properties/) babel plugin as class-tansform is currently an experimental feature and therefore doesn't come with the es2015 or react preset.

```
npm install --save-dev babel-plugin-transform-class-properties
```

And update `.babelrc` to include the new plugin
{% highlight javascript %}
//...
"plugins": ["transform-class-properties"]
//...
{% endhighlight %}

The methods can now be declared like so
{% highlight javascript %}
//...
_onTitleUpdate = (event) => {
  this.setState({ title: event.target.value });
}

_onValueUpdate = (event) => {
  this.setState({ value: event.target.value });
}
//...
{% endhighlight %}

We can then remove both instances of `.bind(this)` where we pass the methods to the `onChange` props.
{% highlight javascript %}
//...
<input type="text" value={this.state.title} onChange={this._onTitleUpdate}/>
//...
<textarea value={this.state.value} onChange={this._onValueUpdate}/>
//...
{% endhighlight %}

Rebuild and run your application and it should still be working as before.

## Displaying our Note
Now we can update the values in our form, we need to do something when the form is submit. Because I want to display the saved value of the note separately from the form for the note, we'll need to create it as a new component. But first we'll create a component that knows about both my current `NoteForm` component and the new display component. We'll create `noteContainer.js` and create a `NoteContainer` component.
{% highlight javascript %}
import React, { Component } from 'react';
import NoteForm from './noteForm.js';

class NoteContainer extends Component{
  constructor(props){
    super(props);
    this.state = {
      note: {
        title: '',
        value: ''
      }
    };
  }

  _onNoteSaved = (note) => {
    this.setState({ note });
  }

  render(){
    return (
      <NoteForm onSave={this._onNoteSaved} />
    );
  }
}

export default NoteContainer;
{% endhighlight %}
Here we've defined a state for our component that consists of a `note` property, this gets updated when the `_onNoteSaved` function gets called. The render function renders a `NoteForm` component, and also also passes `_onNoteSaved` method in a prop we've called `onSave`. In order for this to be called, we'll have to update our `NoteForm` components implementation.


{% highlight javascript %}
//...
_onSubmit = (event) => {
  event.preventDefault();
  this.props.onSave({ title : this.state.title, value: this.state.value });
}

render(){
  return (
    <form onSubmit={this._onSubmit}>
      <div>
        <label>Title:</label>
        <input type="text" value={this.state.title} onChange={this._onTitleUpdate}/>
      </div>
      <div>
        <label>Note:</label>
        <textarea value={this.state.value} onChange={this._onValueUpdate}/>
      </div>
      <input type="submit" value="Save"/>
    </form>
  );
}
//..  
{% endhighlight %}

We have now included a button to submit the form, and included the `onSubmit` prop with a new function as its value, on the `<form>` component. The new `_onSubmit` function takes the event, the first thing we need to do is stop the default behaviour of that event from happening (this would be the browser trying to handle the form submission), we next call the function that was passed to us as the `onSave` prop from the `NoteContainer`. If you remember `onSave` referred to a function that expected a `note` parameter. Therefore we are passing an object with both our title and value properties (the shape of `note`).

Before we create a component to display the saved values of our note, we'll make sure our application still works as we intend, however because we want our `NoteContainer` component to be rendered we'll have to update our `App` component slightly.

{% highlight javascript %}
//...
import NoteContainer from './noteContainer';

 const App = () => {
   return (
     <div>
      <h1>My React App</h1>
      <NoteContainer/>
     </div>

  );
};
//...
{% endhighlight %}

If you build and run your application now, you should still have your form, and clicking on save appears to do nothing!
<img src="/assets/2017-03-19-react-basic-components/note-form.gif" title="React Note Form">

We can now create our component to display our note. I'll create a component called `Note` in `note.js`. Because this components only job will be to render data that is passed to it, we can create this as a stateless functional component, we'll make it expect a `note` prop, and it will display the `note.title` and `note.value`.

{% highlight javascript %}
import React from 'react';

const Note = ({ note }) => {
  return(
    <div>
      <h2>{note.title}</h2>
      <p>{note.value}</p>
    </div>
  );
};

export default Note;
{% endhighlight %}

Now we can update `NoteContainer` to display our new `Note` component as well as the form. We'll make sure that we pass the `note` in the state down to our new `Note` componont in a `note` prop.

{% highlight javascript %}
render(){
  return (
    <div>
      <NoteForm onSave={this._onNoteSaved} />
      <Note note={this.state.note} />
    </div>
  );
}
{% endhighlight %}

If you build and run your application now, when you save your note, you should see it get displayed below!
<img src="/assets/2017-03-19-react-basic-components/note-display.gif" title="React Note Display">

You should now have a grasp of creating components with React both as ES6 classes and stateless functional components, and the differences between Container and Presentation components. Next time we'll look at introducing redux to make managing state much easier!

Find the source code for this over at [github here](https://github.com/airzy91/react-redux-starter/tree/basic-components).
