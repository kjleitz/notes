# JQuery notes

---

[toc]

---

## Basics

`$` is just a function, equivalent to `jQuery`, sort of a fancy alias of `document.querySelectorAll()`. Use it to select w/ CSS-style selectors.

## Selectors

Seems like you can use powerful CSS selectors (mostly pseudoelements) I didn't even know existed, and ones that are "experimental" and are not supported in browsers. Check out `:contains`, `:has`, `:not`, etc. Wow. Very cool.

## Manipulating the DOM

### Append and prepend

Use `append()` and `prepend()`.

```js
$('#some_id').append("yo, what up");
$('p.par-for-editing').prepend("This is now the first sentence!");
```

You can also use `elementToAppend.appendTo(someElementSelector)` which seems nice, look it up.

### Before and after

Use `before()` and `after()`:

```js
$('p.par-for-editing').before("<p>A preceding paragraph.</p>");
$('p.par-for-editing').after("<p>A following paragraph.</p>");
```

### Changing inner HTML

Easy peasy, use `html()`:

```js
$('div.some_class').html("This is now the text inside these divs");
```

### Hiding elements

Easy again, use `hide()`:

```js
// hide all paragraphs which have <i> tags in them
$('p:has(i)').hide();
```

### Showing hidden elements

Guess. You were right! It's `show()`:

```js
$('p:has(i)').hide();
$('p:has(i)').html("what up, <i>girl?</i>").show();
```

## Changing element attributes

### Arbitrary attributes

Use `attr()` sorta like you would `css()` (in reading/writing).

### Adding classes

Use `addClass()`:

```js
$('p:has(i)').addClass('has-italics');
```

### Toggling classes

Use `toggleClass()`, self-explanatory.

### Changing value

Easy, nice, awesome, whoa man slow down this is all so great, use the `val()` method:

```js
$('input[type="text"].required').val("This field is required!");
```

## Changing styles

### Arbitrary CSS editing

Editing CSS directly is easy, use `css()`:

```js
// target some id'd elements
$('#some_id').css('width', 500);
$('#some_id').css('width', 'auto');

// target all anchors
$('a').css('text-decoration', 'none');

// use multiple values for an attr by using an object
$('#some_id').css({
  "background": "url('background.png') repeat-y"
});

// multiple attrs with object literal as well
$('#some_id').css({
  "background": "url('background.png') repeat-y",
  "border": "1px solid gray"
});
```

You can also _get_ css with the same method:

```js
$('#some_id').css('border') // returns border style
```

### Editing specific CSS attributes

You can use, for example, `width()` and `height()` to directly alter an attribute:

```js
$('#some_id').width(500);
$('#some_id').height(150);
```

## Event listeners

### Arbitrary listeners

Use `on()`. Kinda cool. You can pass data on the event when it occurs, too (or you can leave that parameter out):

```js
function identifyElem(e) {
  alert(e.data.message);
}

var somethingData = {
  message: "The 'something' element was clicked"
}

var somethingElseData = {
  message: "The 'something else' element was clicked"
}

$('#something').on('click', somethingData, identifyElem);
$('#somethingElse').on('click', somethingElseData, identifyElem);
```

### Ready

Add an event listener for document ready with `ready()`.

```js
$('document').ready(function() {
  //blah
});
```

### Click

```js
$('p.removable').click(function(){
  $(this).remove();
});

$('p.replaceable').click(function(){
  $(this).replaceWith("<p>yo this is new</p>");
});
```

### Mouseover + Mouseout

```js
$('p.hoverable').mouseover(function(){
  $(this).css('background-color', 'blue')
});

$('p.hoverable').mouseout(function(){
  $(this).css('background-color', 'red')
});
```

### Or, just hover (for both)

```js
$('p.hoverable').hover(function(){
  $(this).css('background-color', 'blue')
}, function(){
  $(this).css('background-color', 'red')
});
```

### Doubleclick

Use `dblclick()` the same way.

### Mousemove

Same thing. Use `mousemove()`.

### Input stuff

Some fun events for input:

```js
// when unfocused
$('input[type="text"]').blur(function(){
  // blah
})

// when in focus
$('input[type="text"]').focus(function(){
  // blah
})

// when value changes
$('input[type="text"]').change(function(){
  // blah
})

// when value is highlighted
$('input[type="text"]').selected(function(){
  // blah
})
```

### Events

Pass `e` (or whatever) into your callback as a param, use fun stuff like this:

```js
$('#something').click(function(e) {
  e.pageX;
  e.pageY;
})

$('#something').movemove(function(e) {
  e.screenX;
  e.screenY;
})
```

## Animations

### Arbitrary animations

Use `animate()`:

```js
$('#something').click(function() {
  $(this).css('position', 'relative');
  
  $(this).animate({
    left: 300,
    opacity: 0.5,
    'font-size': '22px',
    width: 300
  }, 2000, 'easeInQuad', function() {
    alert("Animation finished!");
  });
});
```

### Hiding/showing animations

#### Fadein + Fadeout + FadeToggle

Use these like you would `hide()`, only give it a time in milliseconds to fade. Seems like you can also give fade functions a string like `'slow'` instead.

#### FadeTo

Use `fadeTo(<time>, <opacity between 0 and 1>)`.

#### SlideUp

Same thing, give a time. It'll slide up and hide.

## Components

You pass each of these (most if not all) an object with properties set to various things. Look into this when you actually need to use them, but just know they're there.

### Menus

Use the `menu()` function. Worth checking out. I don't care enough to get into this here.

### Accordions

Use the `accordion()` function. Worth checking out. I don't care enough to get into this here.

### Tabs

Use the `tabs()` function. Worth checking out. I don't care enough to get into this here.

### Dialog boxes

Use the `dialog()` function. Worth checking out. I don't care enough to get into this here.

### Select menu

Use the `selectmenu()` function. Worth checking out. I don't care enough to get into this here.

### Date Picker

Use the `datepicker()` function. Worth checking out. I don't care enough to get into this here. Opens up a calender.

### Tooltips

Use the `tooltip()` function. Doesn't need an object passed to it. Little hovering tooltip on mouse hover.

### Draggable and Droppable

Use `draggable()` and `droppable()` to make stuff drag-and-drop. Check 'em out.

## Iterating

### Using `each`

Use `each` _sorta_ like `forEach`?

```js
$('p').each(function(index) {
  $(#some_div).append($(this).text());
})
```

## AJAX

Say you set it up so clicking on one of three buttons will perform some function that uses AJAX to do something with the server:

```js
$(document).ready(function() {

  $('#btn1').click(getTextFromServer);
  $('#btn2').click(getDoubledNumberFromServer);
  $('#btn3').click(getXMLFromServer);

});
```

Each of those functions might use AJAX like this:

```js
function getTextFromServer() {
  $.ajax({
    type:    "GET",
    url:     "text_from_server.txt",
    success: putOnPageText
  });
}

function getDoubledNumberFromServer() {
  // find out how to call a script on the server, screw php
}

function getXMLFromServer() {
  $.ajax({
    type:     "GET",
    url:      "xml_from_server.xml",
    datatype: "xml",
    success:  putOnPageXML
  });
}

function putOnPageText(data, status) {
  $('#display-area').text(data);
}

function putOnPageXML(data, status) {
  // deal with XML, blech
}
```