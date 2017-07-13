# Chrome Dev Tools notes

---

[toc]

---

## Breakpoints

### Debugger

You can ad the JS keyword `debugger` in the code and you'll have the dev tools stop at that moment for execution at that point, like a breakpoint.

### Breakpoint at a line in the script

You can set a breakpoint at a line in a script, and step through function calls and stuff. Super useful.

### Breakpoint on an element (!!!)

In the elements inspector, you can right click and do "Break on..." > "Subtree Modifications", "Attributes Modifications", "Node Removal", and you can see/step through functions WHEN THAT HAPPENS. Holy SHIT, dude.

#### Subtree modifications

Whenever _anything_ in the subtree changes. Super awesome.

### Pause On Caught Exceptions

There's a little stop sign icon with a pause symbol inside (`||`) which you can click to pause on exceptions. You also have the option to enable "Pause On Caught Exceptions" (which lets you restrict the pausing to only _caught_ exceptions... don't know the difference). It lets you set, essentially, a breakpoint where an error occurs! That's _awesome!_

### Using `debug()` on a method to pause there

Use `debug(someMethod)` to pause when it's used. Super useful. 

## Tips for once you're stopped

### Call stack

See what called what in the Call Stack pane.

### Step through code

Use the arrow buttons to step through the code or call stack.

### Async

There's a little "Async" checkbox nezt to the Call Stack pane. If you don't select it, if you have a function that is called asynchronously, you won't get much info from the call stack (it will not have much context aside from the other asynchronously called code). If you _do_ select it, you get a call stack that actually shows you where it came from! Sweet!

### Scope Variables

See local, global, and even closure variables in the Scope Variables pane!

## Monitoring method calls and events

Oh. Em. Gee. Use `monitor(someMethod)` to get a log of all the uses of that method and their arguments, whenever they're triggered. Amazing. For example:

Marionette views use the `trigger()` function to call all events. So, do this:

```js
> // in the console
> monitor(Marionette.View.prototype.trigger)
```

...then click/interact with whatever thing is going to trigger events, and you'll see (depending on your code, of course):

```
...
function Backbone.Events.trigger called with arguments: item:rendered, [object Object]
function Backbone.Events.trigger called with arguments: before:show, [object Object]
function Backbone.Events.trigger called with arguments: before:show
function Backbone.Events.trigger called with arguments: show, [object Object]
function Backbone.Events.trigger called with arguments: show
function Backbone.Events.trigger called with arguments: dom:refresh
... etc.
```

How fucking cool is that?
