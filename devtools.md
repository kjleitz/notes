# Chrome Dev Tools notes

---

[toc]

---

## Debugger

You can ad the js keyword `debugger` in the code and you'll have the dev tools stop at that moment for execution at that point, like a breakpoint.

## Breakpoints

### Breakpoint at a line in the script

You can set a breakpoint at a line in a script, and step through function calls and stuff. Super useful.

### Breakpoint on an element (!!!)

In the elements inspector, you can right click and do "Break on..." > "Subtree Modifications", "Attributes Modifications", "Node Removal", and you can see/step through functions WHEN THAT HAPPENS. Holy SHIT, dude.

#### Subtree modifications

Whenever _anything_ in the subtree changes. Super awesome.

## Async

There's a little "Async" checkbox nezt to the Call Stack pane. If you don't select it, if you have a function that is called asynchronously, you won't get much info from the call stack (it will not have much context aside from the other asynchronously called code). If you _do_ select it, you get a call stack that actually shows you where it came from! Sweet!

## Pause On Caught Exceptions

There's a little stop sign icon with a pause symbol inside (`||`) which you can click and get the option to enable "Pause On Caught Exceptions". That lets you set, essentially, a breakpoint where an error occurs! That's _awesome!_

