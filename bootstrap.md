#Bootstrap notes


##Setup 

1. You use a `.container` for all your content, which centers it. (`.container-fluid` stretches the width to 100%)  

1. Inside the `.container`, you have some `.row` elements, for each row of content.

1. Inside each `.row`, you have columns, which are `.col-sm-4` for example.

1. Remember, Bootstrap is mobile-up, and media queries should be used as such

##Spacing columns

###Offset

~~~html
<div class="col-sm-offset-4 col-sm-4">
~~~

to add a spacer of 4 columns (above small device width, of course) before the element that has this class.

###Push and pull 

~~~html
<div class="col-sm-push-4 col-sm-4">
~~~

to push your column 4 columns to the right, or 
 
~~~html
<div class="col-sm-pull-4 col-sm-4">
~~~

to pull 4 columns to the left (to rearrange columns, for example).

When you push and pull, the cells of the columns you manipulate will overlap cells that were previously there, unless you push/pull the other cells accordingly.

###Nesting columns

You can nest rows within columns, and those rows will be able to be occupied by columns based on an interior 12 col system. Simple nesting.

##Images

###Responsive

You can use the `.img-responsive` class to automatically set `max-width: 100%;` and `height: auto;` for the image, which will scale it to its parent element.

`.img-responsive` doesn't set `width: 100%;` though, which you might want (so that the image _fills_ the parent element, not just "won't exceed" the parent... at least I'm pretty sure that's the only reason. Avi has a comment potentially indicating `width` is for IE and `max width` is for the others, but that doesn't make much sense to me), so you can open up your general CSS stylesheet and do one of these:

```css
img {
  width: 100%;
}
```
(see _"Images overflowing?"_ in the [CSS notes](css.md))

##Components and plugins

Holy balls are there a lot of nice pre-made components on the [website](http://getbootstrap.com/components).

###Components

There are plenty of pre-made navbars, icons, toolbars, button groups, media embeds, progress bars, alerts, panels, input, you name it.

###Plugins

You can include JS plugins by simply including bootstrap.js (or bootstrap.min.js). see [here](http://getbootstrap.com/javascript). Modals, drop menus, scrolling updates to menus, tabs, tooltips, sidebar page progress, image carousels, and more. They look good, too.

##Media queries and mobile-up

Really want to design with mobile first, then use progressive enhancement to make the site look better on desktops. So, you can do this kind of thing:

```css
/* how your site will be styled by default (with small
devices) */

.carousel-caption p {
  display: none;
}

/* start displaying more/adding to layout as size
steps up */

@media only screen and (min-width: 768px) {

  .carousel-caption p {
    display: block;
  }
}

/* and more... */

@media only screen and (min-width: 992px) {

  etc {
    ...
  }
}

/* and more. */

@media only screen and (min-width: 1200px) {

  etc {
    ...
  }
}
```

You can find the specific breakpoint widths for bootstrap's sm, md, lg, etc., [here](http://getbootstrap.com/css/#grid-media-queries).

