# Backbone notes

A lot of this material is taken from [this great talk](https://www.youtube.com/watch?v=PcTVQyrWSSs).

## Models

### Creating a model

Make a model like this:

```js
var Wine = Backbone.Model.extend();

var cabernet = new Wine({
    name: "Cabernet Sauvignon"
});
```

## Collections

### Creating a collection

Make a collection of models:

```js
var Wines = Backbone.Collection.extend({
    Model: Wine,
    url: '#'
});
```

Gotta set a `url` (to associate with the API server-side) because backbone tries to be RESTful.

### Using a collection

```js
var wines = new Wines([
    { name: "Malbec" },
    { name: "Pinot Noir" }
]);

wines.each(function(model) {
    console.log(model.get('name'));
});
```

## Views

### Creating a view

```js
var HomeView = Backbone.View.extend({});
```

Has a property, `el`, which refers to its element (a div unattached to the DOM by default, or whatever you want). You can render stuff into it.

It looks kinda like this:

```js
{
    $el: /* jquery fn for `el` */,
    cid: "view0",
    el: "div",
    options: Object
}
```

### Rendering in a view

You can render this shiz with a `render()` function on the view object. Return `this` at the end so you can chain function calls after it.

```js
var HomeView = Backbone.View.extend({
    render: function() {
        this.$el.append("<h1>Home</h1>");

        return this;
    }
});
```

But you also want to have it render on initialization:

```js
var HomeView = Backbone.View.extend({
    initialize: function() {
        this.render();
    },

    render: function() {
        this.$el.append("<h1>Home</h1>");

        return this;
    }
});
```

But, hey! That's still not going to actually _render_ anything until you attach it to the DOM and properly display it:

```js
var HomeView = Backbone.View.extend({
    
    el: 'body',

    initialize: function() {
        this.render();
    },

    render: function() {
        this.$el.clear();
        this.$el.append("<h1>Home</h1>");

        return this;
    }
});
```

Noice. Pretty easy, right?

### Using views

Maybe you want to create a list:

```js
var ListView = Backbone.View.extend({

    tagName: 'ul',

    initialize: function() {
        
    },

    render: function() {
        this.$el.clear;
        this.$el.append("<li>List item #1</li>");
        this.$el.append("<li>List item #2</li>");

        return this;
    }
});
```

If you notice, the `ListView` doesn't have an attachment to the DOM yet (up there it's specifying `tagName` not `el`; that overrides the initial `'div'` `el` type). It also doesn't immediately render in the `initialize()` function. Now, back in your `HomeView` you might do something like this:

```js
var HomeView = Backbone.View.extend({
    
    el: 'body',

    initialize: function() {
        this.render();
    },

    render: function() {
        this.$el.clear();
        this.$el.append("<h1>Home</h1>");

        this.listView = new ListView();
        this.$el.append(this.listView.render().el);

        return this;
    }
});
```

Check it out. You create a new `ListView` view object, then `render()` it so you get the element all created (which is stored in its `el`), then append its `el` to the `HomeView`'s element. Useful chaining!

## Routing

### Starting the router

You can start a router in Backbone like so (assuming you have created `AppRouter` like in the next section):

```js
$(document).ready(function() {
    var wineApp = new AppRouter();
    Backbone.history.start();
})
```

Starting the history allows you to go forwards and backwards in the browser.

### Creating the router & routes

Create the router:

```js
var AppRouter = Backbone.Router.extend({
    
    routes: {
             "": "home",
          "add": "add",
        "close": "home"
    }

    home: function() {
        console.log("home");
        new HomeView();
    }

    add: function() {
        console.log("add");
        new AddView();
    }
});
```

Not sure beyond that, but probably need to know it.