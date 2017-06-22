# Redux notes

## Reducers

Pure function, takes a `state` and `action` and returns a modified state (not mutated). Use a switch on the `action.type` to return new state. Export that function.

```js
// given some action === {
//   type: 'INCREASE',
//   incAmt: 2
// }

// src/reducers/managePresents.js

export function managePresents(state, action) {
  switch (action.type) {
    case 'INCREASE':
      return {...state, numberOfPresents: state.numberOfPresents + action.incAmt};
    default:
      return state;
  }
}
```

Doesn't persist that state yet, though. In an abstract sense, we might have this pattern going on so that we can persist state and render it to the page when it happens:

```js
let state = {count: 0};
 
function changeState(state, action){
    switch (action.type) {
 
      case 'INCREASE_COUNT':
        return { count: state.count + 1 }
 
      default:
        return state;
    }
  }
 
function dispatch(action){
    state = changeState(state, action)
    render()
}
 
function render(){
    document.innerHTML = state.counter
}
```

(this stuff is taken from [this Learn lesson](https://github.com/learn-co-students/redux-initial-dispatch-v-000))

Great, renders the number every time `dispatch()` is called with an action. But it never shows `0`... so we might call `dispatch()` with an initiation action first, like so:

```js
dispatch({ type: '@@INIT' })
```

Ah. There we go. Doesn't hit the switch statement, just renders. A type of `@@INIT` is the convention.

Also, we don't want to have state explicitly declared as a global up there, we want it to be managed by the reducer. So, we can declare it without defining it, then pass a default value to `changeState()` (yay ES6!) to set the initial state:

```js
let state;
 
function changeState(state = { count: 0 }, action) {
    switch (action.type) {
 
      case 'INCREASE_COUNT':
        return { count: state.count + 1 }
 
      default:
        return state;
    }
  }
 
function dispatch(action){
    state = changeState(state, action)
    render()
}
 
function render(){
    document.innerHTML = state.count
}
 
dispatch({ type: '@@INIT' })
```

...now this happens:

```js
    dispatch({ type: '@@INIT' })
        -> { counter: 0 }
    dispatch({ type: 'INCREASE' })
        -> { counter: 1 }
```

Woo! Think about it... the first time we call `dispatch` with the init action, it does this:

```js
//=> state === undefined
changeState(undefined, { type: '@@INIT' })
//=> state === { count: 0 }
```

Cool. With this design, we can have the state start off undefined, and initialize with some initial state as a behavior of the reducer.

Now, attach a listener on a button (for example) that calls dispatch with the appropriate action, and you've got yourself a pizza pie.

[This Learn lesson](https://github.com/learn-co-students/redux-dispatch-with-event-listeners-v-000) is a great overview of the Redux `action -> reducer -> new state` / `action -> dispatch -> reducer -> render` pattern.

## Store pattern

So, you wanna make sure `state` isn't overwritten somewhere in your application? You want to make stuff modular so other parts of your application can use it properly? Well, _NOW YOU CAN!_

Encapsulate the `state` global and `dispatch()` function in a function itself, create a getter for `state` so that only `dispatch()` can modify it, then return the two functions:

```js
function changeCount(state = { count: 0 }, action) {
  switch (action.type) {
    case 'INCREASE_COUNT':
      return { count: state.count + 1 };
    default:
      return state;
  }
};

function createStore() {
  let state;
  
  function getState() {
    return state;
  }
  
  function dispatch(action) {
    state = changeCount(state, action);
    render();
  }
  
  return {
    getState,
    dispatch
  }
}

function render() {
  document.getElementById('main').innerHTML = state.count; // or whatever
}

const store = createStore();

store.dispatch({ type: '@@INIT' });
```

That's awesome. But hey, you still need the rest of the application to use a store, with other reducers than `changeCount()`, so you don't want to hardcode the reducer in there... Pass it in as an argument to the `createStore()` function instead!

```js
// ...

function createStore(reducer) {
  let state;
  
  function getState() {
    return state;
  }
  
  function dispatch(action) {
    state = reducer(state, action);
    render();
  }
  
  return {
    getState,
    dispatch
  }
}

// ...

const store = createStore(changeCount);
const button = document.getElementById('button');
 
button.addEventListener('click', function() {
  store.dispatch({ type: 'INCREASE_COUNT' });
});
```

Cool. [Good Learn article on this store pattern](https://github.com/learn-co-students/redux-create-store-v-000).