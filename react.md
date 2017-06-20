# React notes

---

[toc]

---

## The basics

React is love. React is life.

### Setting it up

Add the React + ReactDOM libraries to your `index.html`, as well as your script:

```html
<!-- ... -->
<main id="app">
</main>

<script src="node_modules/react/dist/react.js"></script>
<script src="node_modules/react-dom/dist/react-dom.js"></script>

<script src="index.js"></script>
<!-- ... -->
```

(after page content, before your script)

Then, in `index.js`, grab the element you want to render into by its `id` with `document.getElementById()`, and render some React element or whatever into it with `ReactDOM.render()`:

```js
const app = document.getElementById('app');
const heading = React.createElement('h1', {}, "Yo, what UP");
ReactDOM.render(heading, app);
```

## Creating elements (non-JSX)

Check it:

```js
const boldText = React.createElement('b', {}, "I'm some text, but BOLD!");
const par = React.createElement('p', {className: 'my-par'}, boldText);
```

Tag name, props object, children (pro tip: can be an array). You don't have to do this manually (you can use JSX), but it's good to know. You can also pass in a component instead of an element or string or whatever (see [here](#components) for component stuff) like this:

```js
// say you have a component called Button...
React.createElement(
  'div',
  {},
  [
    React.createElement(Button),
    React.createElement(Button)
  ]
)
```

Just made a `div` with two buttons in it. Easy as pie.

<a name="components">
## Components
</a>

### Creating components (old-school)

This is outdated, but still. `React.createClass()` takes an argument that is essentially the spec for your component, and it's required to have a `render()` method:

```js
const Button = React.createClass({
  render() {
    return React.createElement('button', {}, 'Click me!');
  }
});

const ShoppingList = React.createClass({
  render() {
    return React.createElement('ul', {}, [
      React.createElement('li', {}, 'Bananas'),
      React.createElement('li', {}, 'Vanilla ice cream'),
      React.createElement('li', {}, 'Chocolate'),
    ]);
  }
});
```

Components are always capitalized!

### Creating components (new-school)

ES6 class syntax, baby:

```js
class Button extends React.Component {
  render() {
    return React.createElement('button', {}, 'Click me!');
  }
}
```

You can, of course, add other methods to the component, but here we're just adding the [required] `render()` method.

## JSX

### The basics

JSX allows you to write XML in your JavaScript files. WhoOoOoOa! Check it out:

```jsx
class Tweet extends React.Component {
  render() {
    return (
      <div className="tweet">
        <img src="http://twitter.com/some-avatar.png" className="tweet__avatar" />
        <div className="tweet__body">
            <p>We are writing this tweet in JSX. Holy moly!</p>  
        </div>
      </div>
    );
  }
}
```

(from [this Learn lesson](https://github.com/learn-co-students/react-jsx-v-000))

Dem class attrs gots to be `className`. Also, `for` is another keyword, so use `htmlFor` instead. Furthermore, you can only return _one_ element (that element can have children, but only one top-level element allowed).

## Browserify

(see the [browserify notes](browserify.md) for more info)

Use browserify to transform JSX and `import`s and whatever into JavaScript that is readable by browsers.

Install it:

```
$ npm install --save-dev browserify
```

### Compile `imports`

Compile your code:

```
$ browserify index.js -o bundle.js
```

First is our main script (which imports everything else), the `-o` target is the output file.

If browserify is not globally installed, it will complain. But, that's okay, we'll compile with an `npm` script instead. In your `package.json`:

```json
// ...
"scripts": {
  // ... (other scripts)
  "bundle": "browserify index.js -o bundle.js",
}
// ...
```

You can run that script with:

```
$ npm run bundle
```

`npm` knows to look for commands in the `scripts` in `node_modules/.bin/` before looking system-wide, so it should work just fine.

### Transforming JSX with Babel

(see the [babel notes](babel.md) for more info)

Gotta use the right transformer to translate JSX correctly into readable JavaScript.

First, install some stuff:

```
$ npm install --save-dev babel-core babel-preset-es2015 babel-preset-react babelify
```

- `babel-core` – babel itself, no plugins (does nothing)
- `babel-preset-es2015` – a 'preset' bundle of transformer plugins that transform ES2015 syntax into universal JS
- `babel-preset-react` – another preset, this one for turning JSX (and React stuff in general, I guess?) into universal JS
- `babelify` – transformer for Browserify

Now, tell Babel what presets to use in a config file, `.babelrc`, in the root of your project:

```json
{
  "presets": ["es2015", "react"]
}
```

Tell Browserify what transformer to use:

```json
// ...
"scripts": {
  // ... (other scripts)
  "bundle": "browserify index.js -t babelify -o bundle.js",
}
// ...
```

Cool. Compile your code:

```
$ npm run bundle
```

Awesome! It should have transpiled into one file, `bundle.js`.

**REMEMBER!** Always `import React from 'react'` when using JSX. Babel uses `React.createElement()` to turn JSX into JS, so if you don't, it won't work.

Finally, reference the _bundled_ code in `index.html`:

```html
<!-- ... -->
<script src="bundle.js"></script>
<!-- ... -->
```

## Writing modules for `import`

### The basics

We'll want to write components in their own files. Make a `components/oo.js` and do this:

```js
export const message = "I am a component!";
```

That's a named export! A module can export multiple things that way, and import them like this in, say, `index.js`:

```js
import { message } from './components/foo'
```

Remember to use a relative path like that, even in the same directory, otherwise it'll look in `node_modules` or globally.

### Named exports

In addition to the previously-mentioned example, you can export multiple named things at once like this, too:

```js
// In a file called `fruits.js`
export default {
  apple: 'red',
  banana: 'yellow',
};
```

```js
// In a file in the same directory
import fruits from './fruits'
console.log(fruits.apple); // 'red'
```

```js
// In another file, also in the same directory
import { apple } from './fruits'
console.log(apple); // 'red'
```

Cool, right?

### Default export

Often you want to export only one thing, like if you have components in separate files:

```jsx
// In a file called `Tweet.js`
import React from 'react'
 
class Tweet extends React.Component {
  render() {
    return (
      <div className="tweet">
        <img src="http://twitter.com/some-avatar.png" className="tweet__avatar" />
        <div className="tweet__body">
            <p>We're writing this tweet in JSX. Holy moly!</p>  
        </div>
      </div>
    );
  }
}

export default Tweet;
```

If I'm not mistaken, I think you can even reduce that to, like, `export default class Tweet extends...` right there at the class definition. Then:

```jsx
// In a file in the same directory
import Tweet from './Tweet'
import ReactDOM from 'react-dom'

ReactDOM.render(
  <Tweet />,
  document.getElementById('main')
);
```

Noice.

## Props

### defaultProps

Check it [out, yo](https://facebook.github.io/react/docs/typechecking-with-proptypes.html). Use `propTypes` to ensure your props are the proper types, and use it to assign default values to props if one is not assigned.

Use default props like this:

```jsx
import React from 'react';

export default class Spaceship extends React.Component {
  render() {
    return (
      <div>
        <p>{this.props.name}</p>
        <p>{this.props.speed}</p>
        <p>{this.props.hasRockets}</p>
        <p>{this.props.colors}</p>
      </div>
    );
  }
}

Spaceship.defaultProps = {
  hasRockets: false,
  colors: ['black', 'red']
};
```

### propTypes

#### The basics

Prop type checking is cool: throws errors in development (not production) if props do not pass validations. You can ensure a prop is present, ensure a prop is a string, etc. Use prop types like this:

(taken from [the docs](https://facebook.github.io/react/docs/typechecking-with-proptypes.html))

```jsx
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // You can declare that a prop is a specific JS primitive. By default, these
  // are all optional.
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // Anything that can be rendered: numbers, strings, elements or an array
  // (or fragment) containing these types.
  optionalNode: PropTypes.node,

  // A React element.
  optionalElement: PropTypes.element,

  // You can also declare that a prop is an instance of a class. This uses
  // JS's instanceof operator.
  optionalMessage: PropTypes.instanceOf(Message),

  // You can ensure that your prop is limited to specific values by treating
  // it as an enum.
  optionalEnum: PropTypes.oneOf(['News', 'Photos'])
};
```

Note: the `propTypes` property on the component starts with a lowercase `p`, while the `PropTypes` class which holds the validator types starts with a capital `P`.

There are more things you can do, but those are the basics. See more at the documentation link. Also note: `React.PropTypes` is deprecated as of React v15.5, which is why you need to `import PropTypes from 'prop-types';`.

#### Array of type 'blah'

Check this out, too, validating that something is an array of strings:

```jsx
MyComponent.propTypes = {
  someArray: PropTypes.arrayOf(PropTypes.string)
};
```

#### Object of shape 'blah'

Also, how about instead of just validating an object like this:

```jsx
Order.propTypes = {
  orderInfo: PropTypes.object.isRequired
};
```

...you can use `shape` to specify how the object should be structured:

```jsx
Order.propTypes = {
  orderInfo: PropTypes.shape({
    customerName: PropTypes.string.isRequired,
    orderedAt: PropTypes.number.isRequired // We're using UNIX timestamps here
  }).isRequired
};
```

#### Range

Here's a range pattern I came up with (validates that a weight is a number in a range between 80-300):

```js
//...
  weight: PropTypes.oneOf(Array(301-80).fill(80).map((e, i) => e + i)
//...
```

#### Custom validators

This validates the same thing with a custom validator:

```js
//...
  weight: function(props, propName, componentName) {
    const weight = props[propName];
    if (typeof weight !== 'number' || weight > 300 || weight < 80) {
      return new Error('Invalid ' + propName + 'supplied to ' + componentName + '. Validation failed.');
    }
  }
//...
```

Cool.

### Children

#### Accessing `this.props.children`

If you have child nodes rendered within a component, like this:

```jsx
<Panel title="Browse for movies">
  <div>Movie stuff...</div>
  <div>Movie stuff...</div>
  <div>Movie stuff...</div>
  <div>Movie stuff...</div>
</Panel>
```

...how do you actually render the child nodes inside the `Panel` component? You access those nodes through `this.props.children`:

```jsx
export default class Panel extends React.Component {
  render() {
    return (
      <div className="panel">
        <div className="panel-header">{this.props.title}</div>
        <div className="panel-body">{this.props.children}</div>
      </div>
    );
  }
}
```

If you didn't have `this.props.children` you'd have to pass all those into the component `Panel` as a prop value... that'd be nuts.

#### Iterating over children

The children could potentially be `undefined` if none, an `Element` if one, or an `Array` if multiple. So, as you could imagine, it can get hairy checking to see if you are able to iterate over them. React supplies a pretty clean way to do this. You might want to take this component:

```jsx
<MovieBrowser>
  <Movie title="Mad Max: Fury Road" />
  <Movie title="Harry Potter & The Goblet Of Fire" />
</MovieBrowser>
```

...and let each of those `Movie` components know if they are the currently-playing movie with a boolean prop. Previously, your `MovieBrowser` component might have looked like this:

```jsx
export default class MovieBrowser extends React.Component {
  render() {
    const currentPlayingTitle = 'Mad Max: Fury Road';
 
    return (
      <div className="movie-browser">
        {this.props.children}
      </div>      
    );
  }
}
```

...but you can now add the prop to the children like this:

```jsx
export default class MovieBrowser extends React.Component {
  render() {
    const currentPlayingTitle = 'Mad Max: Fury Road';
    const childrenWithExtraProp = React.Children.map(this.props.children, child => {
      return React.cloneElement(child, {
        isPlaying: child.props.title === currentPlayingTitle
      });
    });
 
    return (
      <div className="movie-browser">
        {childrenWithExtraProp}
      </div>      
    );
  }
}
```

...using `React.Children.map()` and `React.cloneElement()`! No data mutation. `React.cloneElement()` is like `React.createElement()`; you pass it the element you wish to clone, and a set of props that will get merged into the cloned element's props.

## State

### Setting state

State can change. Props can't be changed directly. Set initial state in the `constructor()` method like so:

```jsx
class ToggleButton extends React.Component {
  constructor(props) {
    super(props);
 
    this.state = {
      isEnabled: false
    };
  }
 
  render() {
    return (
      <div className="toggle-button">
        I am toggled {this.state.isEnabled ? 'on' : 'off'}.
      </div>
    );
  }
}
```

Keep state small. Have as little data in the state as possible. State should be used sparingly.

### Changing state

#### Use `setState()`

Do not modify the state directly! Only modify the state directly when you are setting its initial value in the constructor.

e.g., this will not re-render a component:

```js
// bad
this.state.comment = 'Hello';
```

...but this will:

```js
// good
this.setState({comment: 'Hello'});
```

See [Using State Correctly](https://facebook.github.io/react/docs/state-and-lifecycle.html#using-state-correctly).

#### Nested state will get overwritten!

The `setState()` function _merges_ the object you pass in into the `this.state` object. But it only does it one level deep. So, with an initial state of:

```js
{
  theme: 'blue',
  addressInfo: {
    street: null,
    number: null,
    city: null,
    country: null
  },
}
```

...and a `setState()` call like this:

```js
this.setState({
  addressInfo: {
    city: 'New York City',
  },
});
```

...you would get a `this.state` of:

```js
{
  theme: 'blue',
  addressInfo: {
    city: 'New York City',
  },
}
```

Ouch. Instead of doing that, you have two options. One, use `Object.assign()`:

```js
this.setState({
  addressInfo: Object.assign({}, this.state.addressInfo, {
    city: 'New York City',
  }),
});
```

...to manually merge that deeper layer. You also could use the object spread operator (but you gotta enable that plugin in Babel, at the moment):

```js
this.setState({
  addressInfo: {
    ...this.state.addressInfo,
    city: 'New York City',
  },
});
```

That would be really nice, wouldn't it? But stick with `Object.assign` for now, I guess.

#### Setting state is asynchronous

React batches state updates. Don't expect to be using the new state immediately after setting it:

```js
handleClick() {
  this.setState({
    hasBeenClicked: true,
  });
  console.log(this.state.hasBeenClicked); // prints false
}
```

If you need something to happen after the state has been set, you can pass an optional callback function to `setState()`:

```js
handleClick() {
  this.setState({
    hasBeenClicked: true
  }, function () {
    console.log(this.state.hasBeenClicked); // prints true
  });
}
```

## Events

React has its own event system (`SyntheticEvent`), which act pretty much the same as normal JS events (same interface: `stopPropagation()` and `preventDefault()` work, etc.), but they are an abstracted API so that React can provide a consistent interface to events across all browsers. Check [here](https://facebook.github.io/react/docs/events.html#supported-events) for a list of events.

### Event handlers

You can register handlers for an event by assigning a function to an inline handler, like a prop:

```jsx
class Tickler extends React.Component {
  constructor() {
    super();
 
    this.tickle = this.tickle.bind(this);
  }
 
  tickle() {
    console.log('Tee hee!');
  }
 
  render() {
    return (
      <button onClick={this.tickle}>Tickle me!</button>
    );
  }
}
```

They always start with `on` and then the description of the event, in camelCase (e.g., `onClick`, `onKeyUp`, `onMouseDown`, `onFocus`, `onSubmit`, etc.). Pass it a reference to a function (named or anynoymous) and you're good to go.

### Binding methods

You're gonna want to bind `this` on methods created on component classes. Take this example, from the docs on the [differences when not using ES6 React syntax](https://facebook.github.io/react/docs/react-without-es6.html#autobinding) (the overall topic is obviously tangential):

```jsx
class SayHello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {message: 'Hello!'};
    // This line is important!
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    alert(this.state.message);
  }

  render() {
    // Because `this.handleClick` is bound, we can use it as an event handler.
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
}
```

Remember to bind your methods. I think that looks like a decent pattern.

### Asynchronous access to events

Events can't be accessed asynchronously, as they are [pooled](https://facebook.github.io/react/docs/events.html#event-pooling) (I don't know _exactly_ what that means, but that link does a good job of explaining the implications). You need to store a reference to the properties or whatever you're accessing on the event, because it will be nullified.

**Edit:** Ah, okay, so pooling refers to this:

> Event pooling means that whenever an event fires, its event data (an object) is sent to the callback. The object is then immediately cleaned up for later use. This is what we mean by 'pooling': the event object is in effect being sent back to the pool for use in a later event.
> 
> _(from [this Learn lesson](https://github.com/learn-co-students/react-events-in-detail-v-000))_

The solution to pooling nullifying the event object is either to store a reference in a variable (`const target = event.target`) or to use `event.persist()`.

## Forms

### Differences between HTML and React JSX

See [this helpful documentation](https://facebook.github.io/react/docs/forms.html).

### Controlled and uncontrolled components

Basically, a controlled component has its `value` prop set (e.g., on an `<input />`). React controls that component. If it doesn't have a `value` prop, it's uncontrolled, and may optionally have a `defaultValue` prop to set its initial value.

In an uncontrolled component, its state is maintained in the DOM itself, and to retrieve that value, we'd either need access to the component itself or use an `onChange` handler on the component.

In a controlled component, we're setting the value directly with React, so we need to update the value with any changes that the user actually makes:

```jsx
class ControlledInput extends React.Component {
  constructor() {
    super();
 
    this.handleChange = this.handleChange.bind(this);
 
    this.state = {
      value: '',
    };
  }
 
  handleChange(event) {
    this.setState({
      value: event.target.value,
    });
  }
 
  render() {
    return (
      <input type="text" value={this.state.value} onChange={this.handleChange} />
    );
  }
}
```

Seems weird, but this is the preferred way of doing things, because we manage that component's state _entirely_ in the React state, and we don't go back and forth from the DOM to access that value through it's internal state. Consider the following:

> It might seem a little counterintuitive that we need to be so verbose, but this actually opens the door to additional functionality. For example, let's say we want to write an input that only takes in a number (let's pretend there is no `<input type="number">`). We can now validate the data the user enters before we set it on the state, allowing us to block any invalid values. If the input is invalid, we simply avoid updating the state, preventing the input from updating. We could optionally set another state property (for example, `isInvalidNumber`). Using that state property, we can show an error in our component to indicate that the user tried to enter an invalid value.
> 
> If we tried to do this using an uncontrolled component, the input would be entered regardless, since we don't have control over the internal state of the input. In our `onChange` handler, we'd have to roll the input back to its previous value, which is pretty tedious!
> 
> _(from [this Learn lesson](https://github.com/learn-co-students/react-forms-v-000))_

## Component Lifecycle

This is mostly taken from [this Learn lesson](https://github.com/learn-co-students/react-component-lifecycle-v-000). Has a good table for comparing utility of each hook. Also, check out the docs [here](https://facebook.github.io/react/docs/react-component.html) for a more complete lifecycle reference. It's pretty dece, yo.

### Create stage

Components need to be created (and rendered). Some hooks will be called during that process.

#### `componentWillMount()`

Occurs just _before_ `render()`; use it to change the initial state if necessary. Calling `this.setState` in this method will not trigger a re-render.

#### `componentDidMount()`

Occurs just _after_ `render()`; use it to set up expensive processes like fetching/updating data.

### Update stage

When state or props change, a component is re-rendered. Some hooks will be called during that process.

#### `componentWillReceiveProps(nextProps)`

This is called when the component's props (which it has received from a parent) change and trigger a re-render. You could use this to change state based on the new props. Apparently the props aren't always _actually_ different, so don't assume so until you've checked.

#### `shouldComponentUpdate(nextProps, nextState) => Boolean`

Called just _before_ a component re-renders. You can prevent an unnecessary re-render here by comparing props and state and whatever. Return a boolean to decide.

#### `componentWillUpdate(nextProps, nextState)`

Called just after `shouldComponentUpdate` and just before `render`. Useually you use this method to update integrations with third party libraries. Cannot use `setState` in this method.

#### `render()`

Obvious. Required.

#### `componentDidUpdate(prevProps, prevState)`

Called just _after_ the re-render. Use it to update any third party libraries if they happen to need an update due to the re-render. Receives previous props and state as arguments! That seems pretteh useful, since you have access to the new ones in `this.state` and `this.props`.

### Delete stage

#### `componentWillUnmount()`

Gets called just before a component is deleted. Use it to tear down anything you set up in `componentWillMount()`.

> For example, if you had a component that displays the weather data in your home town, you might have set it up to re-fetch the updated weather information every 10 seconds in `componentDidMount`. When the component gets deleted, you wouldn't want to continue doing this data-fetching, so you'd have to get rid of what was set up in `componentWillUnmount`.

## Separation of concerns

### Presentational vs. container components

Keep presentational (entirely UI) components small and stateless. Nothing but a render. Put logic in a container component (created for that precise purpose), and pass data into presentational components through props. For event handling in presentational components, use callbacks (passed in as props) to call functions from a container.

See [here](https://css-tricks.com/learning-react-container-components/).