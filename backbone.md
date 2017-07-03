# Backbone notes

A lot of this material is taken from [this great talk](https://www.youtube.com/watch?v=PcTVQyrWSSs). Most of the rest is taken from the [backbonejs docs](http://backbonejs.org) and [Backbone, The Primer](https://github.com/jashkenas/backbone/wiki/Backbone%2C-The-Primer).

## Models

Models manage internal tables of data attributes, and trigger `"change"` events when any of their data are modified. They sync data with a persistence layer (usually some kind of REST API attached to a database).

### Creating a model

Make a model like this:

```js
var Wine = Backbone.Model.extend();

var cabernet = new Wine({
    name: "Cabernet Sauvignon"
});
```

In a hypothetical scenario with muppets where the API looks like this:

```
GET  /muppets/ ...... Reads all Muppets.
POST /muppets/ ...... Creates a new Muppet.
GET  /muppets/:id ... Reads a Muppet.
PUT  /muppets/:id ... Updates a Muppet.
DEL  /muppets/:id ... Destroys a Muppet.
```

...and a `GET /muppets` returns the following data structure:

```js
{
  "total": 2,
  "page": 1,
  "perPage": 10,
  "muppets": [
    {
      "id": 1,
      "name": "Kermit",
      "occupation": "being green"
    },
    {
      "id": 2,
      "name": "Gonzo",
      "occupation": "plumber"
    }
  ]
}
```

(those nested muppets being full muppet model data)

...you might build a Kermit model like this:

```js
var KermitModel = Backbone.Model.extend({
  url: '/muppets/1',
  defaults: {
    id: null,
    name: null,
    occupation: null
  }
});
```

Normally you'd have a model as part of a collection, and it would auto-construct the `url` as `"<collection.url>/<model.id>"`.

### Using models

You might then build an instance of a `KermitModel` and do stuff with it:

```js
var kermit = new KermitModel();

kermit.fetch().then(function() {
  kermit.get('name'); // >> "Kermit"
  kermit.get('occupation'); // >> "being green"
  kermit.set('occupation', 'muppet ringleader');
  kermit.save();
});
```

### RESTful methods on models

Check it out:

- `fetch`: fetches the model's data from its REST service using a GET request.
- `get`: gets a named attribute from the model.
- `set`: sets values for named model attributes (without persisting).
- `save`: sets attributes, and then persists the model data using a PUT request.
- `destroy`: decommissions the model, and persists this using a DELETE request.


## Collections

Collections are groups of related models. Handles the loading/saving of new models to the server. Provides helper functions for performing aggregations/computations against lists of models. Collections have their own events, but also proxy through all the events that occur on models within them, so you can listen in _one place_ for changes that happen to models inside.

### Creating a collection

Make a collection of models:

```js
var Wines = Backbone.Collection.extend({
    Model: Wine,
    url: '/wines'
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

### REST resources with collections and models

Check it out! (directly from the backbone docs):

>The Collection and Model components together form a direct mapping of REST resources using the following methods:
>
>```
>GET  /books/ .... collection.fetch();
>POST /books/ .... collection.create();
>GET  /books/1 ... model.fetch();
>PUT  /books/1 ... model.save();
>DEL  /books/1 ... model.destroy();
>```

To continue the muppet metaphor, say you define a muppet model and attach the model class to a muppet collection:

```js
var MuppetModel = Backbone.Model.extend({
  defaults: {
    id: null,
    name: null,
    occupation: null
  }
});
  
var MuppetsCollection = Backbone.Collection.extend({
  url: '/muppets',
  model: MuppetModel
});
```

This collection will load data from the `/muppets` endpoint, then construct the data into a collection of `MuppetModel` instances:

```js
var muppets = new MuppetsCollection();

muppets.fetch().then(function() {
  console.log(muppets.length); // => length: 1
});
```

Great! One problem, though: the JSON returned from the API endpoint is actually an object with some metadata at the root and a `muppets` collection _attached,_ like this:

```js
{
  "total": 2,
  "page": 1,
  "perPage": 10,
  "muppets": [
    {
      "id": 1,
      "name": "Kermit",
      "occupation": "being green"
    },
    {
      "id": 2,
      "name": "Gonzo",
      "occupation": "plumber"
    }
  ]
}
```

...which is why it only creates _one_ muppet (it creates a "muppet" out of the entire object). So, you can override the `parse()` method on the collection class:

```js
var MuppetCollection = Backbone.Collection.extend({
    url: '/muppets',
    model: MuppetModel,

    parse: function(data) {
        return data.muppets;
    }
});
```

Easy. It takes an argument of the data you get back, and whatever it returns becomes your data collection.

Sweet. Now it's usable:

```js
var muppets = new MuppetCollection();

muppets.fetch().then(function() {
  console.log(muppets.length); // => length: 2
});

muppets.get(1); // => Returns the "Kermit" model, by id reference
muppets.get(2); // => Returns the "Gonzo" model, by id reference
muppets.at(0);  // => Returns the "Kermit" model, by index
muppets.findWhere({name: 'Gonzo'}); // => returns the "Gonzo" model
```

Useful methods:

- `fetch`: fetches the collection's data using a GET request.
- `create`: adds a new model into the collection, and persists it via POST.
- `add`: adds a new model into the collection without persisting.
- `remove`: removes a model from the collection without persisting.
- `get`: gets a model from the collection by id reference.
- `at`: gets a model from the collection by index.
- `find`: finds all records matching specific search criteria.

## Views

Views are atomic chunks of UI. They usually render data from a specific model (or number of models), but they can also just be lone data-less chunks of UI. Models are unaware of the associated views. Views listen to the model "change" events and react/re-render accordingly.

### Creating a view

```js
var HomeView = Backbone.View.extend({});
```

Has a property, `el`, which refers to its element (a div unattached to the DOM by default, or whatever you want) which is actually a selector string (like, it could be `'#muppets-list'` to target a `<ul id="muppets-list"></ul>` somewhere!). You can render stuff into it.

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
    className: 'my-list',

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