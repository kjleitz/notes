# JavaScript notes

---

[toc]

---

## mininotes

### Template literals

Use backticks (`` ` ``) to use templates in a string; you can insert variables or evaluate js by enclosing them ``var inAString = `like ${this()}` ``

### Find keys of object / whether it is empty or not

Use `Object.keys(someObject)` to return an array of keys (properties) of your object, which is useful for determining if an object is empty

### Types

```js
"2" + 2
//=> "22" // (and vice versa) 

typeof 1
//=> "number"

typeof 1.1
//=> "number"

typeof "ha"
//=> "string"

typeof true
//=> "boolean"

typeof null
//=> "object"

Object.keys(null)
//=> Uncaught TypeError: Cannot convert undefined or null to object
//    at Function.keys (<anonymous>)
//    at <anonymous>:1:7
```

## Functions

blah blah blah arrow functions anonymous functions you know all that.

### Using `bind()`

You can use `bind()` to return a new function with a supplied `this` context and pre-set arguments. It's actually pretty useful. For example (contrived but whatever):

```js
function multiply(value1, value2) {
  return value1 * value2;
}

var double = multiplier.bind(null, 2);
var triple = multiplier.bind(null, 3);
```

See more [here](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind), but you should look up some videos and get yourself used to this, because this kind of metaprogramming always comes in super handy in Ruby.

### Using `call()` and `apply()`

Pretty sure `call()` is roughly the same as `bind()`, but it actually calls the function. Useful if you're calling a function as a callback within another function, since the callback has closure over the function and might therefore have a different context for `this` than intended (could be global scope if it's a global function it's being called in, could be the object the function is a method of, etc.). 

You can use `apply()` to do the same thing... the only difference being that `call()` takes a value to be defined as `this`, then all subsequent arguments are applied as arguments to the function, whereas `apply()` takes a `this` value, then an _array_ of values to be supplied as arguments to the function.

See the following excerpt from [that badgood website](http://javascriptissexy.com/understand-javascript-callback-functions-and-use-them/):

>When the callback function is a method that uses the this object, we have to modify how we execute the callback function to preserve the this object context. Or else the this object will either point to the global window object (in the browser), if callback was passed to a global function. Or it will point to the object of the containing method.
>Let’s explore this in code:
>
>```js
// Define an object with some properties and a method​
​// We will later pass the method as a callback function to another function​
​var clientData = {
    id: 094545,
    fullName: "Not Set",
    // setUserName is a method on the clientData object​
    setUserName: function (firstName, lastName)  {
        // this refers to the fullName property in this object​
      this.fullName = firstName + " " + lastName;
    }
}
​
​function getUserInput(firstName, lastName, callback)  {
    // Do other stuff to validate firstName/lastName here​
​
    // Now save the names​
    callback (firstName, lastName);
}
```

>In the following code example, when clientData.setUserName is executed, this.fullName will not set the fullName property on the clientData object. Instead, it will set fullName on the window object, since getUserInput is a global function. This happens because the this object in the global function points to the window object.

>```js
getUserInput ("Barack", "Obama", clientData.setUserName);
​
console.log (clientData.fullName);// Not Set​
​
​// The fullName property was initialized on the window object​
console.log (window.fullName); // Barack Obama
```

Good example. Bad author. Bad author! Why is that site so badgood?

If I'm not mistaken, you can use arrow functions (which bind `this` to the scope so that it applies to the context where it was defined—bad explanation, look it up) to make this less of a problem (but sometimes you want this behavior, obviously). I'll have to write a section on closures, too, I guess.

### This (`this`)

`this` is the execution context (whatever called the function, basically). Basically, the scope where the function was called. It could be an object, an event, another function, etc. A reference to `this` in a function called in the global scope returns the `window` object.

About strict mode, from Learn.co:

>In strict mode, `this` remains at whatever it's set to when entering the execution context. If it's not defined, it remains `undefined` rather than being assigned the default `window` object.

We can also use `this` within an event handler to refer to the element that triggered the event.

## Console

Log some stuff:

```js
console.log("some stuff")
console.log("can do", "multiple strings", "separated by", "spaces")
```

Log an error:

```js
console.error("some error")
console.error("can do", "multiple strings", "too")
```

Log a warning:

```js
console.warn('Hm, you might not want to do that.')
```

## Require-ing libraries

```js
const expect = require('expect')
const fs = require('fs')
const jsdom = require('jsdom')
const path = require('path')
```

## Functions

Function hoisting only works with this syntax:

```js
console.log(square)    //=> prints the function
console.log(square(5)) //=> 25

function square(n) {
    return n * n
}
```

Not this syntax:

```js
console.log(square)    //=> undefined
console.log(square(5)) //=> TypeError: square is not a function

var square = function() {
    return n * n
}
```

## Incrementor/decrementor (++/--)

`x++` increments after returning the value:

```js
var number = 5

number++  //=> 5
number    //=> 6

number--  //=> 6
number    //=> 5
```

`++x` increments before returning the value:

```js
var number = 5

++number  //=> 6
number    //=> 6

--number  //=> 5
number    //=> 5
```

## Parsing strings

### parseInt(string, base)

Parse a string to return an integer:

```js
parseInt('2', 10)      //=> 2
parseInt('2.1', 10)    //=> 2
parseInt('2.9', 10)    //=> 2
parseInt('2blah', 10)  //=> 2
parseInt('2blah3', 10) //=> 2
parseInt('blah', 10)   //=> NaN
parseInt('b2lah', 10)  //=> NaN
```

### parseFloat(string)

Parse a string to return a float:

```js
parseFloat('80.123999') //=> 80.123999
```

## Arrow functions

### Arguments

You can omit parens if there is only one argument (I like it better with explicit parens but okay):

```js
var multiply = (n, m) => { return n * m; }

var square = n => { return n * n; }
```

### Implicit return

You can implicitly return by omitting braces:

```js
var square1 = n => n * n
var square2 = n => { n * n }
var square3 = n => { return n * n }

square1(5)
//=> 25

square2(5)
//=> undefined

square3(5)
//=> 25
```

### Anonymous

Arrow functions are always anonymous because they have no identifier:

```js
function iHaveAName() {}
 
iHaveAName.name // 'iHaveAName'

var anonymous = () => {}
 
anonymous.name // ''
```

## Hoisting

### With `var`

Variable declarations with `var` are hoisted to the "top" of the "document"(/scope), but assignments are not. So, this:

```js
var someFunction = function() {
    // stuff
};

var someVariable = 1;
```

...is basically this:

```js
var someFunction, someVariable;

somefunction = function() {
    // stuff
}

someVariable = 1;
```

So, if you access the variable before assignment, its value will be `undefined`.

### With `let` and `const`

Technically, they're both hoisted, but you can't access them before assignment (it will throw an error instead of returning `undefined`). From MDN:

> In ECMAScript 2015, let [and const] will hoist the variable to the top of the block. However, referencing the variable in the block before the variable declaration results in a ReferenceError. The variable is in a "temporal dead zone" from the start of the block until the declaration is processed.
> 
> [MDN - let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let)

### Function declarations vs. function expressions

If you declare a named function (as opposed to a function expression where you set some `var` equal to a function), the entire function will hoist:

```js
eat(); // time to eat cake!

function eat() {
    console.log("time to eat cake!");
}
```

...as opposed to:

```js
eat(); //=> Uncaught TypeError: eat is not a function

var eat = function() {
    console.log("time to eat cake!");
};
```

## Arrays

### Mutation array manipulation

#### Adding to the array

- `push()` adds to the end
    - returns new `length` property of the array
- `unshift()` adds to the beginning
    - returns new `length` property of the array

#### Removing from the array

- `pop()` removes from the end
    - returns removed element (`undefined` if empty)
- `shift()` removes from the beginning
    - returns removed element (`undefined` if empty)

#### Replacing elements in the array

`splice(index, num_elems_to_remove, ...elems_to_add)` will remove/replace/add items to an array and return the removed elements:

```js
var items

// remove everything after index 1 (inclusive):

items = [1, 2, 3, 4]

items.splice(1)
//=> [2, 3, 4]

items
//=> [1]


// at index 1, remove 1 item:

items = [1, 2, 3, 4]

items.splice(1, 1)
//=> [2]

items
//=> [1, 3, 4]


// at index 1, remove 1 item and add 6 and add 7 there:

items = [1, 2, 3, 4]

items.splice(1, 1, 6, 7)
//=> [2]

items
//=> [1, 6, 7, 3, 4]
```

### Non-mutation array manipulation

#### Adding with the "spread" operator: `...` (coooool!)

The `...` operator (supported in most modern browsers) spreads an array out in place:

```js
var cities = ["New York", "San Francisco"]

["Philadelphia", ...cities]
//=> ["Philadelphia", "New York", "San Francisco"]

cities
//=> ["New York", "San Francisco"]

[...cities, "Philadelphia"]
//=> ["New York", "San Francisco", "Philadelphia"]
```

#### Removing with `slice()`

`slice(start_index, end_index=-1)` returns a section of the array that you specify with the start and end indices (or, just specify start and it will go to the end):

```js
var cats = ["Milo", "Garfield", "Otis"]

cats.slice(1)
//=> ["Garfield", "Otis"]

cats
//=> ["Milo", "Garfield", "Otis"]
```

#### Replacing/replicating `splice()` with `slice()` and `...`

```js
var items = [1, 2, 3, 4, 5]

// removing the third element:

[...items.slice(0, 2), ...items.slice(3)]
//=> [1, 2, 4, 5]
```

#### Map, reduce, filter (and other filter functions)

- use `map()` as you would `map`/`collect` in Ruby
- use `reduce()` as you would `reduce`/`inject` in Ruby
- use `filter()` as you would `select` in Ruby
- use `find()` as you would `find`/`detect` in Ruby

From the [AirBnB style guide](https://github.com/airbnb/javascript#iterators-and-generators):

> Use `map(`) / `every()` / `filter()` / `find()` / `findIndex()` / `reduce()` / `some()` / ... to iterate over arrays, and `Object.keys()` / `Object.values()` / `Object.entries()` to produce arrays so you can iterate over objects.

## Objects

Construct an object like this:

```js
var meals = {
    breakfast: "cereal",
    lunch: "pizza",
    dinner: "spaghetti"
}

// or:

var meals = new Object({
    breakfast: "cereal",
    lunch: "pizza",
    dinner: "spaghetti"
})
```

### Keys

#### They're strings!

Object keys are always strings, even if you initialize an object with what look like barewords (`breakfast:` in the previous example is actually `"breakfast":`).

#### Using variables as object keys

ES6 allows you to use a variable as an object key by wrapping it in `[]`:

```js
var firstMeal = "breakfast"
var secondMeal = "lunch"
var thirdMeal = "dinner"

var meals = {
    [firstMeal]: 'cereal',
    [secondMeal]: 'pizza',
    [thirdMeal]: 'spaghetti'
}

// same as above!! Cooooooool.
```

### Values

#### Reading

You can access the values using array notation (`someObject["someKey"]`) or dot notation `someObject.someKey`.

#### Writing

##### Mutation

You can do:

```js
someObject.someKey = "something"

// or:

someObject["someKey"] = "something"
```

##### Non-mutation

You can also use `Object.assign(...objects)` to "merge" or "replace" properties into existing objects without affecting the original object (!!! this is awesome!):

```js
Object.assign({}, { foo: 'bar' })
//=> { foo: 'bar' }
 
Object.assign({ eggs: 3 }, { flour: '1 cup' })
//=> { eggs: 3, flour: '1 cup' }
 
Object.assign({ eggs: 3 }, { chocolate: '1 cup', flour: '2 cups' }, { flour: '1/2 cup' })
//=> { eggs: 3, chocolate: '1 cup', flour: '1/2 cup' }
```

Very cool.

Apparently you can also use the spread (`...`) operator to accomplish the same thing (and the AirBnB style guide [recommends it](https://github.com/airbnb/javascript#objects--rest-spread)).

#### Deleting

Delete a key/value pair with `delete` keyword:

```js
var meals = { breakfast: "oatmeal", lunch: "turkey sandwich", dinner: "steak and potatoes" };

// the `delete` operator returns `true` if it has successfully
// deleted, `false` otherwise
delete meals.dinner;
//=> true

meals;
//=> { breakfast: "oatmeal", lunch: "turkey sandwich" }
```

### Duplicating an object

Looks like the best way to do this is like so:

```js
a = {a: "A"}
//=> {a: "A"}

b = Object.assign({}, a)
//=> {a: "A"}

a === b
//=> false
```

Check out this other shit, too:

```js
//------------------------------------------------------
// Two object literals are not the same object:

{a: "A"} === {a: "A"}
//=> false

//------------------------------------------------------
// "a" references the same object:

a = {a: "A"}
//=> Object {a: "A"}

a === a
//=> true

//------------------------------------------------------
// "b" references a different object with same keys/values:

b = {a: "A"}
//=> Object {a: "A"}

a === b
//=> false

//------------------------------------------------------
// "c" references "a" so they reference the same object:

c = a

a === c
//=> true

c === b
//=> false

//------------------------------------------------------
// "d" is a new object out of "a" but it's the same object:

d = new Object(a)
//=> Object {a: "A"}

a === d
//=> true

//------------------------------------------------------
// "e" uses assign to merge just "a", and it's the same:

e = Object.assign(a)
//=> Object {a: "A"}

a === e
//=> true

//------------------------------------------------------
// "f" uses assign and an empty literal to DUPLICATE!!!:

f = Object.assign({}, a)
//=> Object {a: "A"}

a === f
//=> false
```

### Iterating over an object

#### With `for...in`

You can use a `for...in` loop to iterate over the properties of an object. You can also do this with an array, but it will return the _keys_ of the array (which are the indices, not the elements), as it is little more than an object itself.

```js
var obj = {a: 1, b: 2, c: 3};
    
for (let prop in obj) {
  console.log('obj.' + prop, '=', obj[prop]);
}

// "obj.a = 1"
// "obj.b = 2"
// "obj.c = 3"
```

You can check with `hasOwnProperty()` to make sure properties are of the inherited object (wording?) before manipulating them:

```js
var triangle = {a: 1, b: 2, c: 3};

function ColoredTriangle() {
  this.color = 'red';
}

ColoredTriangle.prototype = triangle;

var obj = new ColoredTriangle();

for (let prop in obj) {
  if (obj.hasOwnProperty(prop)) {
    console.log('obj.' + prop + ' = ' + obj[prop]);
  } 
}

// "obj.color = red"
```

#### With `for...of`

Holy cow! How'd I never know this? You can iterate over the _values_ of the properties of an object with `for...of` (the same way you would iterate over the _properties_ of an object with `for...in`). You can use this on any iterable collection, like arrays, too. Check it [out](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of).

## Math

- Use `Math.random()` to make a value between 0 (inclusive) and 1 (non-inclusive).
- Use `Math.floor(n)` to round down to nearest integer
- Use `Math.ceil(n)` to round up to nearest integer

## Algorithms

### Breadth-First Search

Maybe this shouldn't be here, but I wrote a recursive algorithm from scratch, not having seen a solution before, to perform a [Breadth-First Search](https://en.wikipedia.org/wiki/Breadth-first_search) (extra good job patting yourself on the back for this one, Keegs). I'm proud of it! I did it before reading the one in the Learn readme for this lesson and... and... I wanna keep it and I think it's good for posterity (it's probably bad code... but it was hard to reason about! And mine uses no mutation!):

```js
const ary = [11, 12, [21, 22, [31, 32, 33], 23], [[34, 35], 24, [36]], 13];

function findWith(array, criteriaFn) {
  const firstMember = array[0];
  
  if (criteriaFn(firstMember)) return firstMember;
  if (typeof firstMember === "undefined") return null;
  
  if (Array.isArray(firstMember)) {
    const result = findWith(firstMember, criteriaFn);
    if (result) return result;
  }

  return findWith(array.slice(1), criteriaFn);
}

// works for all members of ary
findWith(ary, (n) => n === 34); //=> 34
findWith(ary, (n) => n === 12); //=> 12
findWith(ary, (n) => n === 24); //=> 24
findWith(ary, (n) => n === 50); //=> null
```

Here's the one from Learn (minus comments):

```js
function find(array, criteriaFn) {
  let current = array
  let next = []

  while (current) {
    if (criteriaFn(current)) {
      return current
    }
 
    if (Array.isArray(current)) {
      for (let i = 0, l = current.length; i < l; i++) {
        next.push(current[i])
      }
    }

    current = next.shift()
  }

  return null
}
```

...which I believe could be improved by:

```js
function find(array, criteriaFn) {
  let current = array
  const next = []

  while (current) {
    if (criteriaFn(current)) return current;
    if (Array.isArray(current)) next.push(...current);
    current = next.shift()
  }

  return null
}
```

But, those require mutation, which would be nice to avoid :)

For posterity, an explanation I gave Alex:

the function, in a broad view, is given an array of arrays of arrays (of arbitrary length and complexity—it’s a tree. every time there’s an array within the array, you can think of that as a branch, and branches can have branches can have branches). Within the function, it goes takes the first member of the array, sees if it matches. If it doesn’t match, it sees if it’s out of members. Every time it fails to find the result, it cuts off the head of the array, and runs again (thereby checking each member of the array by continuously cutting off the first member and checking the first member of the “new” array).

1. Check to see if the first member of the array it’s given matches the criteria… in that case, return that member, because that’s the one we’re looking for!

2. If there _is no first member_ of the array it’s been given (a.k.a. the array is empty, because it’s had its head cut off so many times), return `null` to signify that the current array being searched has been reduced to nothing and no matching member was found

3. If it gets past those without returning, we know that there _is_ a member present, and it _hasn’t_ matched the criteria, so we check to see if the member is an array, itself (a branch with leaves). If it is, we run the function on that branch the same way we ran it on the root array. Same deal. And from the context where it’s called again, right here, it doesn’t matter what’s going on inside the function. All we need to know is that if it finds the member somewhere in that nested array (even if it has branches, itself), it will return that member. If it exhausts all the members of the array, it will return `null`. So if it finds it, then return it, otherwise go on to the final step

4. Run the function again, but without the first member of the array (essentially doing the same thing but with the second member of the array, until the array is exhausted and returns `null`, or the result if it finds it)

So it will search _every member_ of the tree until it finds (and returns) it or returns `null`.

```      
      [11, 12, v, v, 13]
               |  |
              /    \
             ^      ^
 [21, 22, v, 23]  [v, 24, v]
          |        |       \
         /         |        \
        ^          ^         ^
[31, 32, 33]    [34, 35]    [36]
```

…like that. So when you call:

```
findWith(ary, (n) => n === 34);
```

…it goes:

```              
              [no, no, fw, fw: return 34, 13]
                       /   ^ \       ^
   ,------------------`   /   `---,   \
   v                     /        v    \
 [no, no, fw, no: return null]   [fw: return 34, 24, v]
          /    ^                   |  ^               \
         |      `------,           |   \               |
         v              \          v    \              v
       [no, no, no: return null]  [yes: return 34, 35] [36]
```

(`no` means it checked it and no match, `fw` means it called `findWith` on that branch, `return` means it returned something other than continuing with a shortened array)

### Recursion in general

Before we get into recursion in general... here's a factorial function I just wrote! :D This is getting fun! But also, I'm getting really fucking arrogant by saving these.

```js
function factorial(number) {
  return number <= 1 ? 1 : (number * factorial(number - 1));
}
```

So, I looked up some basic stuff about recursion, and found some helpful tips:

1. Start by writing `if`. Why?
    - there is always a (base) case where the function does not call itself
    - there is always a (recursive) case where the function calls itself
2. Handle the simplest case(s)... a.k.a. the base case. Why?
    - the base case requires no looping; it may be complex, but there's nothing wonky going on, and you can define the "final procedure"
    - you're going to return a value
3. Write the recursive case(s)
    1. Write the recursive call. What do you use as the argument?
        - use the "next simplest value/input/state"
    2. Assume the recursive call works. Ask yourself:
        - what is it supposed to do/return? Maybe store that value in a `result` variable
        - how does that help/what can you do (repeatedly) with it to get your result?

### Return deepest nested element

Gah this took forever:

```js
function deepestChild() {
  const element = document.querySelector("#grand-node");
  let children = element.children;

  if (children.length === 0) return element;

  let nextChildren;

  for (;;) {
    nextChildren = [...children].filter(child => child.children.length > 0);

    if (nextChildren.length === 0) {
      return children[0];
    } else {
      children = nextChildren.reduce((acc, child) => [...acc, ...child.children], []);
    }
  }
}
```

From [this lab](https://github.com/learn-co-students/javascript-hide-and-seek-v-000).

## In the browser

### Useful properties and methods

#### On the `window` object

- `innerWidth` - returns the width of the window
- `innerHeight` - returns the height of the window

#### On the `document` object

- `all` - returns all the nodes inside the `document` object
- `contentType` - usually `"text/html"`
- `URL` - returns URL, obv.
- `getElementsByClassName()`, `getElementById()`, etc. - yada yada
- `querySelector()` and `querySelectorAll()` - CSS selectors, powerful (newish)

#### On a node/element

- `appendChild()` - add a node
- `removeChild()` - remove a node
- `attributes` - a list of its attributes
- `removeAttribute()` - remove an attribute from a node
- `style` - view/add/remove/change CSS on that node
- `addEventListener` - listen for some event on an element/node

### Manipulating elements in the DOM

#### Creating elements

Use `var someDiv = document.createElement('div')` (or whatever tag name), then modify its properties to your discretion, and slap it somewhere on the page with `someNode.appendChild(someDiv)`. Kinda like this:

```js
var element = document.createElement('div');

element.innerHTML = 'Hello, DOM!';
element.style.backgroundColor = '#f9f9f9';

document.body.appendChild(element);
```

#### Changing elements

You can edit those elements in place, too:

```js
element.style.textAlign = 'center';
```

The element will align center as soon as the line is fired off.

We can also append elements to it:

```js
var ul = document.createElement('ul');
 
for (let i = 0; i < 3; i++) {
  let li = document.createElement('li');
  li.innerHTML = (i + 1).toString();
  ul.appendChild(li);
}
 
element.appendChild(ul);
```

We can remove children from it, as well:

```js
ul.removeChild(ul.querySelector('li:nth-child(2)'));
```

You gotta remove it from the parent element. But, it looks like if you create an element, it can only be put in one place on the page... so why bother specifying the parent element? Why can't you just remove it straight from the top? Like, `document.removeChild(someDiv)`? Or, since it knows it can be only in one spot, `someDiv.remove()`? Seems arbitrary. It returns the element it removes, so you can do this:

```js
var someLi = ul.removeChild(ul.querySelector('li:nth-child(2)'));
```

...so you still have a reference to the element, but like, why bother? Do you not always have a reference to it in the first place? Maybe not. But it makes sense that you'd be able to do something like this:

```js
var someLi = ul.querySelector('li:nth-child(2)')
ul.removeChild(someLi);
```

Whatever.

Oh okay, this makes more sense: you _can_ use `.remove()` directly on an element. The former must mostly exist to remove elements that match a more generic selector.

### Events

#### Event listeners

stuff

#### Key presses

Ugh, okay so just... do `e.detail || e.which` if you're looking to check a keypress value. I guess. Goddamnit.

## Async requests

### AJAX

See the [Ajax notes](ajax.md) for stuff about Ajax.

### Fetch

There's a new API for async requests that uses promises and a buncha cool stuff: `fetch()`

With fetch, you can do this kind of thing:

```js
fetch('https://api.github.com/repos/jquery/jquery/commits')
  .then(res => res.json())
  .then(json => console.log(json));
```

You can also pass an options object to fetch after the URL (including things like `method: "POST"`, `body: ...`, `headers: {...}`, etc.).

The `then()` method is a neat feature of...

### Promises

...which represent a value that might not be available yet, but will be at some point in the future.

You can chain `then()` calls on it, and the callback you give to `then` gets the result of the previous one as an argument. Confusing? Maybe. Powerful? Apparently!

A `Promise` object, returned by a function like `fetch()`, represents the result of an operation, whenever it occurs. A promise object has a `then()` method which is called when the promise is fulfilled (completed successfully). The `then()` call returns... another promise?... which can be passed to the next `then()`'s callback function as an argument (and so on, and so on, etc.).

### Authorization with `fetch()`

Ugh, just read [this](https://learn.co/tracks/full-stack-web-dev-with-react/advanced-javascript/accessing-remote-apis-with-javascript/javascript-fetch-lab) and the previous one, if you need to do this.

## Object Oriented JavaScript

### Constructor functions

You can use this (old) pattern:

```js
function Sandwich(bread, crust, meat, condiments, veggies, cheese) {
  this.breadType = bread;
  this.crust = crust;
  this.meat = meat;
  this.condiments = condiments;
  this.veggies = veggies;
  this.cheese = cheese;
}

var blt = new Sandwich("white", false, "bacon", "mayo", ["lettuce", "tomato"], "none");
```

Notice the capital first letter. Use `new` so you don't just return `undefined` (since there's no return value).

You can also get the constructor function as a property from the created object:

```js
blt.constructor;
// returns the Sandwich constructor function
```

### Properties

Access properties with either `[<key>]` or `.<key>` syntax:

```js
// These are equivalent:
blt["breadType"];
blt.breadType;
```

### Inheritance with prototypes

#### Non-inheritance method

Say you want to make a `Rectangle` constructor and a `Square` constructor, and you know that a square is basically just a rectangle, so why duplicate code? You might do something like this:

```js
// Rectangle constructor
function Rectangle(width, height) {
  this.width = width;
  this.height = height;
  this.area = function() {
    return this.width * this.height;
  }
  this.perimeter = function() {
    return (this.width + this.height) * 2;
  }
}
 
// Square constructor
function Square(length) {
  Rectangle.call(this, length, length);
}

var rect = new Rectangle(3, 5);
var square = new Square(2);

rect.area();    //=> 15
square.area();  //=> 4
```

Here, you're directly `call`ing the `Rectangle` constructor function within the `Square` constructor, with the `this` context of `Square`. So, the `Rectangle()` call will set the properties/methods to `Square`'s context! Great! Unfortunately, it only copies what `Rectangle` has at the time of instantiation. It doesn't really _inherit_ from `Rectangle`. Future changes to the `Rectangle` prototype won't carry down to `square`.

#### Actual inheritance with prototypes

WATCH THIS: https://www.youtube.com/watch?v=Yvf_kUBZmXg this video explains every dumb confusing thing about prototypal inheritance in JavaScript. SUPER worth it.

Prototypes are neat. Every object has a prototype, and that prototype is essentially a reference to the "parent" object that that object inherits from (which itself has a prototype, and on and on). When you call a method on an object, if it's not found on the object itself, it will be asked of the object's prototype. If it's not there, it continues up the chain until it's found. It's delegated up the chain.

That's a cool concept. It means you can add methods and properties and changes to the prototypal object on the fly, and then even already-instantiated objects will get those changes! You also get the benefit of efficiency; all those methods and properties aren't _copied_ over to the object, they're delegated to the prototype when needed (and they can be changed at runtime!).

We can do this:

```js
function Shape(sides, x, y) {
  this.sides = sides;
  this.x = x;
  this.y = y;
}

function Rectangle(x, y, width, height) {
  //call superclass constructor
  Shape.call(this, 4, x, y);
  //set rectangle values
  this.width = width;
  this.height = height;
}

// set Rectangle prototype to an instance of a Shape
Rectangle.prototype = Object.create(Shape.prototype);
// set Rectangle constructor
Rectangle.prototype.constructor = Rectangle
 
// extend with Rectangle behavior
Rectangle.prototype.area = function() {
  return this.width * this.height;
}
Rectangle.prototype.perimeter = function () {
  return (this.width + this.height) * 2;
}
```

...and get this:

```js
var rect = new Rectangle(1,0,5,3)
var shape = new Shape(3,2,2)
 
console.log(rect.sides);
// 4
console.log(shape.sides);
// 3
 
console.log(rect.width);
// 5
console.log(shape.width);
// undefined
 
console.log(rect.area());
// 15
console.log(shape.area());
// TypeError - no function
 
console.log(rect instanceof Shape);
//true
console.log(shape instanceof Rectangle);
//false
```

...


Is it that the object has a prototype object (which has properties) which has a prototype object (which has properties), and so on?

Or,

Is it that the object has a prototype object (which has properties), which has a constructor function which has a prototype... wait

The object has a `__proto__` property, which is an object with whatever has been added onto the parent's prototype (`Rectangle.prototype.foo = "bar";`, etc.) and a `constructor` method (property?) that is the parent constructor function (`Rectangle`). Now, `Rectangle` has a `__proto__` property, too! But it's of the basic `function`. `Rectangle` _also_ has a `prototype` property, which is the same as the object referenced by the initial object's `__proto__`. I would go on, but it's turtles all the way down after this.

So, a constructor has a prototype. A prototype is an object that `new` creates an object _with_, and within the constructor, `this` refers to that object. (I think this is technically wrong)

An object is created by `new`, and the constructor initializes stuff on it. That object is also given a `__proto__` property, which references the object held by the `prototype` property of the constructor.

The prototype object referenced by the new object's `__proto__` property carries basically three references:

1. `constructor`, a method, which references the constructor function
2. `__proto__`, a property, which references the prototype object that _it_ was created with (in the same way this object was constructed)
3. anything you add to the constructor function's `prototype` through `Rectangle.prototype.size = "large"`, etc.

SO!

1. Constructor function `Rectangle` is defined
2. Constructor function has a `Rectangle.prototype` object
    3. the prototype object has a method called `constructor` which references the constructor function (`Rectangle`)
    4. the prototype object has a property called `__proto__` which references, by default, `Object`
2. You create a new object from that constructor with `var rect = new Rectangle(2, 5)`
    3. This creates a new object called `rect` with properties initialized from the `Rectangle` function call
    4. `rect.__proto__` is assigned to reference `Rectangle`'s prototype object
3. If you change `Rectangle`'s prototype object (e.g. to an object created by `Object.create(Shape.prototype)`), it changes `rect`'s `__proto__` reference to that object

And the basic instruction set for inheritance:

- Define the initial constructor:

```js
function Rectangle(x, y) {
  this.width = width;
  this.length = length;
  this.area = function() {
    return x * y;
  };
}
```

- Define the constructor of what you want to inherit from `Rectangle`, and call the `Rectangle` constructor with its context:

```js
function Square(length) {
  Rectangle.call(this, length, length);
}
```

- Set the child constructor's prototype to the parent's prototype object:

```js
Square.prototype = Object.create(Rectangle.prototype);
```

- That overrides the fact that `Square`'s prototype's _constructor_ should _still_ be `Square` (not `Rectangle`), so make sure you set that:

```js
Square.prototype.constructor = Square;
```

- Now you're all done! Square's prototype still generates `Square`s, and the prototype of _that_ prototype is `Rectangle`, (and on and on until `Object`), so rectangles inherit from all the way up.
- Side note: you can put those last two lines in a generic `inherit(child, parent)` function, or whatever

Okay, one more time, just some more explanation as far as I understand it:

When you do:

```js
var rect = new Rectangle(3, 4);
```

It does this (?):

```js
var rect = {};
// rect: {}

Rectangle.call(rect);
// rect: {
//   width:  3,
//   height: 4
// }

rect.__proto__ = Rectangle.prototype;
// rect: {
//   width:  3,
//   height: 4,
//   __proto__: {
//     constructor: function Rectangle() {...}
//     __proto__: {
//       constructor: function Shape() {...}
//       __proto__: {
//         ...
//       }
//     }
//   }
// }
```

OHHHHHHHHH! So, with `Object.create(Rectangle.prototype)`, you create essentially what you create with `new Rectangle(3, 4)` only you don't run the actual constructor function (picture `rect` in that comment, but without the properties of its own). That's basically just an object with `__proto__`s referring to previous shit, but without its own properties other than `__proto__`, _including_ the `constructor` method! So, you assign, for example, `Square.prototype` to that object, which becomes its prototype (a.k.a. what instantiated `Square`s will have as their `__proto__` property), but since there's no constructor on that prototype, you assign it one (the constructor function itself)!!! You aren't _reassigning_ a constructor method when you do that, it only _had_ a constructor method because it was DELEGATING it to its prototype!!! You're _assigning_ it its OWN constructor method, which means that the parent prototype object still has its own constructor!!!

#### Puttin' stuff on prototypes

Sometimes it's good to throw methods on the prototype instead of within the constructor function itself. That way, instead of all these objects carrying around all these identical functions (imagine Facebook having 1.19 billion user objects o_o), they could all just reference the same method on their prototype. YAY!

### Using `class` "instead of" prototypes

#### Defining a class

In ES6, we get the `class` keyword. Be aware: these are not real classes, they're just syntactic sugar for the prototypal inheritance pattern we just went over. Check it out:

```js
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
 
  sayHello() {
    console.log("Hello, my name is "+ this.name);
  }
}
 
var sarah = new User("Sarah", "sarah@aol.com");
sarah.sayHello();
```

Neat. Use `constructor()` within the class to define the constructor (the same way you'd define the constructor function, previously; sorta like `init` or `initialize`). You can even see that `sayHello()` is actually on the _prototype_, not on the object itself. That's handy!

Notice also that the method definitions **don't** use the `function` keyword.

#### Inheriting with classes (`extends`)

No need to assign the prototype with `Object.create`. If you're making a `class`, you can do this:

```js
class Teacher extends User {
    sayHello() {
      super.sayHello()
      console.log("I am a teacher");
    }
}
 
var t = new Teacher("Tom", "tom@geocities.edu")
t.sayHello()
```

Noice. Notice that you can now use `super` to access the superclass object.

**Also, note: While function definitions are hoisted, class definitions are not.** If you need to use a class, you must define it before use.

#### Methods using `this`

Yo, reminder: value of `this` can be weird.

```js
class Animal { 
  speak() {
    return this;
  }
  static eat() {
    return this;
  }
}

let obj = new Animal();
let speak = obj.speak;

obj.speak();  //=> Animal {}
speak();      //=> undefined

let eat = Animal.eat;

Animal.eat(); //=> class Animal {...}
eat();        //=> undefined
```

Ugh.

#### "Class methods" (`static` methods)

Use `static` to designate what is basically a class method (can't be used on an instance of the class, only on the class itself):

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  static distance(a, b) {
    const dx = a.x - b.x;
    const dy = a.y - b.y;

    return Math.sqrt(dx*dx + dy*dy);
  }
}

const p1 = new Point(5, 5);
const p2 = new Point(10, 10);

console.log(Point.distance(p1, p2));
```

## Useful Libraries

### Lodash

[Lodash](https://lodash.com/) is a nice library for creating javascript templates (using functions to build up the DOM). It's also a really nice library for other stuff, too, so check that shit _out_. Cool functions for iterating over arrays/objects/strings, too. See more in the [lodash notes](lodash.md).

### Handlebars

[Handlebars](http://handlebarsjs.com/) is also awesome. Restricts logic in the view to keep templates clean. See more in the [handlebars notes](handlebars.md).

### Mocha

Test suite. Pretty cool. A little frustrating sometimes. If you're doing Learn.co labs and you want rspec-like documentation formatting output, do `npm test -- --reporter spec`. Good stuff: it's `npm test`, then a `--` to denote that you’re giving the `test` script extra arguments besides the ones it has pre-defined, then `--reporter spec` which are the arguments you supply to mocha to give an rspec-like documentation output! You could (arguably more simply?) do `./node_modules/.bin/mocha --reporter spec` but that’s kind of a pain to type out. You could also probably just install mocha globally with `npm install -g mocha` and then _probably_ do `mocha --reporter spec` which would be ideal, but I don’t want to potentially fuck with things by installing it globally.