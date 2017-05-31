# Babel notes

## The basics

Babel does nothing on its own. Gotta use plugins. Transpiles new JS language features down to ES5!

## Plugins

Here's a [list of available plugins](https://babeljs.io/docs/plugins/).

### Individual

Small dependencies that transform parts of your code, e.g. the `transform-es2015-destructuring` plugin turns this object destructuring:

```js
// Source code
const { foo, bar } = myLib;
```

...into this valid ES5 syntax:

```js
// Gets transformed by the plugin to:
var _myLib = myLib;
var foo = _myLib.foo;
var bar = _myLib.bar;
```

Here are some common ones:

- `babel-plugin-transform-object-rest-spread` – use the spread (`...`) operator on objects the way you can on `Array`s in ES6
- `babel-plugin-transform-class-properties` – we can use class properties to declare methods so you don't have to use `.bind()` in the constructor... I'm not quite sure what that means yet but [check it out I guess](http://babeljs.io/docs/plugins/transform-class-properties/).

### Presets

Collections of plugins (so you don't have to pick and choose a ton of them for whatever) are called presets. Common ones:

- `babel-preset-es2015` – ES6 syntax
- `babel-preset-react` – React stuff

### Installing & using plugins

Install them using `npm`. Then, make a `.babelrc` file in the root of your project to include them like this:

```json
{
  "presets": ["es2015", "react"],
  "plugins": ["an-example-plugin", "another-example-plugin"]
}
```

## Transforming JSX with Babel (from the React notes)

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
