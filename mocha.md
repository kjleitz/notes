# Mocha notes

## The basics

Mocha is a library that lets you write tests for JavaScript. The following is mostly taken from [here](https://github.com/learn-co-students/javascript-intro-to-mocha-v-000).

## Define a suite

Define a suite with `describe()`. The first argument is the name of the suite.

## Test something

Test with `it()`. It takes a string as the first argument (the subject of the test) and a function that contains an assertion that compares your code to the expected outcome.

## Before and after

If we need to run certain code before or after our tests run, we can use the `before()` and `after()` functions. Similarly, if we want to do something before or after every test, we can make use of `beforeEach()` and `afterEach()`.

## Outside the browser

We need to simulate a browser in our testing environment. To do so, we use `jsdom`, which mocks out objects and behaviors as if we were in a browser without actually forcing us to render anything in a browser window.

You can set it up in a `before()`:

```js
global.expect = require('expect');
 
const jsdom = require('jsdom');
const path = require('path');
 
before(function(done) {
  const src = path.resolve(__dirname, '..', 'index.js');
  const babelResult = require('babel-core').transformFileSync(src, {
    presets: ['es2015']
  });
  const html = path.resolve(__dirname, '..', 'index.html');
 
  jsdom.env(html, [], { src: babelResult.code }, (err, window) => {
    if (err) {
      return done(err);
    }
 
    Object.keys(window).forEach(key => {
      global[key] = window[key];
    });
 
    return done();
  });
});
```

## Example use

Like-a dis:

```js
describe('favoriteIceCream', () => {
  it('should return the correct sentence when passed an icecream flavor', () => {
    const result = favoriteIceCream('mint chocolate chip');
    const expectedResult = 'I love mint chocolate chip';
    expect(result).toBe(expectedResult);
  });
});
```

Pretty self-explanatory.

## Why it never worked in the browser for you

Apparently you gotta have `mocha.setup('bdd')` in there, like this:

```html
<script src="mocha.js"></script>
<script src="https://unpkg.com/expect/umd/expect.min.js"></script>
<script>mocha.setup('bdd');</script>
<script src="../index.js"></script>
<script src="intro-test.js"></script>
<script>
  mocha.run();
</script>
```

Then, throwing `debugger` in a function will be pretty useful, while testing in the browser. JS' answer to `pry`.

## Spying with Sinon

### General

Here are some [best practices](https://semaphoreci.com/community/tutorials/best-practices-for-spies-stubs-and-mocks-in-sinon-js).

### Spies

You can use Sinon.js for spying. See [this lesson](https://github.com/learn-co-students/javascript-spies-v-000) for more info.

### Stubs

You can use it to stub out methods (overwrite them so you can see when they're called with certain stuff), which is great for testig async stuff like an AJAX request (see [here](https://github.com/learn-co-students/javascript-mocks-and-stubs-v-000)).

### Mocks

Not exactly sure, too lazy to put effort into understanding right now. See [here](https://github.com/learn-co-students/javascript-mocks-and-stubs-v-000).