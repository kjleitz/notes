#CSS notes


###Box model and width/height

In normal box model (IE is different) width/height are *before* the padding. If you want the IE model, where the width and height encompass the padding (and the border, I think) use `box-sizing: border-box;`

###% scales to parent element

Ahhh!!! Height/width of an element in % scale to the height/width of the parent element… So, this is why you have an issue with 100% height and not 100% width; `body` is by default 100% width of the page, but only as tall as the content. So, if you want to set the height of an element to be 100% of the page, you need to do:

```css
body, html {
  height: 100%;
  min-height: 100%;
}

div {
  /* this is the div you want to extend to the height of the page */
  height: 100%;
}
```


###display (block, inline, etc.) box model

1. Block elements expand full width, match content height.

1. Adjacent block elements’ margins OVERLAP.

1. Adjacent inline-block elements’ margins do not overlap, they COMBINE.

1. Inline elements ignore margin on top and bottom, and adjacent side margins do not overlap, they COMBINE.


### Quick column structure

A way to get decent column structure is to do the following:

```css
/* one column wide */
.col-1 {
  float: left;
  width: 32%;
}

/* two columns wide */
.col-2 {
  float: left;
  width: 66%;
}

/* three columns wide */
.col-3 {
}

[class*="col-"] {
  margin-left: 2%;
}

.first {
  margin-left: 0;
}
```

…then give those classes to elements which you want to be in columns. Use `.first` on the first column in a row. You probably also want to set `* { box-sizing: border-box }` so that padding doesn’t screw up your columns. 

###floated elements collapse `div`s

Also, if a wrapper div only has floating elements in it (only columns), it will collapse. So you want to put a `clearfix` class in any wrapping divs, for this:

```css
.clearfix:after {
  content: ".";
  display: block;
  clear: both;
  visibility: hidden;
  height: 0;
  line-height: 0;
}
```

…but I am sure there’s a prettier way than that.

###Images overflowing?

If your images or inputs or videos or whatever are overflowing from resized containers, you can do this:

```css
img, video, audio, iframe, table, form {
  width: 100%; /* IE */
  max-width: 100%; /* FF, Safari, Chrome */
}
```

…and they will resize with the container.

###Viewport `<meta>` tag

To deal with viewports and devices scaling things and screwing up your responsive design, use this meta tag:

```html
<meta name=“viewport” content=“width=device-width, initial-scale=1.0”>
```
