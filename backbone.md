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

Another more complex implementation, based on the muppet metaphor:

```js
var KermitModel = Backbone.Model.extend({
  url: '/muppets/1',
  defaults: {
    name: '',
    occupation: ''
  }
});

var KermitView = Backbone.View.extend({
  el: '#kermit-view',

  initialize: function() {
    this.listenTo(this.model, 'sync change', this.render);
    this.model.fetch();
    this.render();
  },

  render: function() {
    var html = '<b>Name:</b> ' + this.model.get('name');
    html += ', occupation: ' + this.model.get('occupation');
    this.$el.html(html);
    return this;
  }
});

var kermit = new KermitModel();
var kermitView = new KermitView({model: kermit});
```

That way, you get a re-render on `sync` or `change` events on the model. You also `GET /muppets/1` on initialization, and render it immediately. Cool.

### Rendering with a template

We could refactor that previous example to use an underscore template:

```html
<div id="kermit-view"></div>

<script type="text/template" id="muppet-tmpl">
  <p><a href="/muppets/<%= id %>"><%= name %></a></p>
  <p>Job: <i><%= occupation %></i></p>
</script>

<script>
var KermitModel = Backbone.Model.extend({
  url: '/muppets/1',
  defaults: {
    name: '',
    occupation: ''
  }
});

var KermitView = Backbone.View.extend({
  el: '#kermit-view',
  template: _.template($('#muppet-tmpl').html()),

  initialize: function() {
    this.listenTo(this.model, 'sync change', this.render);
    this.model.fetch();
    this.render();
  },

  render: function() {
    var html = this.template(this.model.toJSON());
    this.$el.html(html);
    return this;
  }
});

var kermit = new KermitModel();
var kermitView = new KermitView({model: kermit});
</script>
```

Notice that this template is cached by the view class. That makes using it a lot faster.

Templates are also used this way with Marionette.js, so this is a common concept.

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

### Attaching models and collections to views

Check it out (from [Backbone, The Primer](https://github.com/jashkenas/backbone/wiki/Backbone%2C-The-Primer)):

>A view is responsible for binding its document element to a model or a collection instance, provided to the view as a constructor option. For example:
>
>```js
>var myModel = new MyModel();
>var myView = new MyView({model: myModel});
>
>// The provided model is attached directly onto the view:
>console.log(myView.model === myModel); // << true
>Attach a model to a view by providing a {model: ...} constructor option:
>
>var KermitModel = Backbone.Model.extend({
>  url: '/muppets/1',
>  defaults: { . . . }
>});
>  
>var MuppetsListItemView = Backbone.View.extend({
>  tagName: 'li',
>  className: 'muppet',
>  
>  initialize: function() {
>    console.log(this.model); // << KermitModel!!
>  }
>});
>  
>// Create Model and View instances:
>var kermitModel = new KermitModel();
>var kermitView = new MuppetsListItemView({model: kermitModel});
>```
>
>Attach a collection to a view by providing a {collection: ...} constructor option:
>
>```js
>var MuppetsModel = Backbone.Model.extend({ . . . });
>  
>var MuppetsCollection = Backbone.Collection.extend({
>  model: MuppetsModel,
>  url: '/muppets'
>});
>  
>var MuppetsListView = Backbone.View.extend({
>  el: '#muppets-list',
>  
>  initialize: function() {
>    console.log(this.collection); // << MuppetsCollection!!
>  }
>});
>
>// Create Collection and View instances:
>var muppetsList = new MuppetsCollection();
>var muppetsView = new MuppetsListView({collection: muppetsList});
>```

You can use the data associated with the view with `this.model` or `this.collection`, simple as that. Noice.

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

## Events

### Backbone.Events (built-in)

Views need to be able to capture UI events, so you can put an `events` object on the view object which maps DOM events (user input) to function names:

```html
<div id="kermit-view">
  <label>Name: <input type="text" name="name" class="name"></label>
  <button class="save">Save</button>
</div>

<script>
var KermitView = Backbone.View.extend({
  el: '#kermit-view',

  events: {
    'change .name': 'onChangeName',
    'click .save': 'onSave'
  },

  onChangeName: function(evt) {
    this.model.set('name', evt.currentTarget.value);
  },

  onSave: function(evt) {
    this.model.save();
  }
});

var kermitView = new KermitView({model: new KermitModel()});
</script>
```

That's... pretty freaking cool. Declare an event trigger as `"<eventName> <selector>"`.

### Backbone.Radio (Marionette)

Backbone.Radio isn't built in, but it's pretty awesome. It lets you make a namespaced "message bus" object (a `Backbone.Radio.channel(channelName)`), on which you can set event listeners and even use the request-response pattern of event/message passing. [Check it out](https://github.com/marionettejs/backbone.radio) (good explanation in the README). Totally worth it.

Things to check out:

- setting a `default` reply
- setting multiple replies with an object
- make multiple requests with spaces
- `replyOnce()` - only reply once
- `stopReplying()` - remove a reply handler (by request name, optionally callback, optionally context)
- `reset()` - remove all handlers from the channel
- channel activity logging

## Marionette

_Much of this section is taken from [this](https://benmccormick.org/marionette-explained/) series._

Marionette is a framework built with Backbone. Here are some highlights:

### Models

#### Serializing data to send to views

By default, Marionette simply calls `toJSON()` on the model to generate the data to send to the view. However, you can override `serializeData()` on the model (WAIT I THINK THIS IS ACTUALLY ON THE VIEW, I AM IDIOT) to do more complicated things:

```js
serializeData: function () {
    var active = this.collection.getActive().length;
    var total = this.collection.length;

    return {
        activeCount: active,
        totalCount: total,
        completedCount: total - active
    };
},
```

The `serializeData()` method is like a "ViewModel" pattern.

### Views

#### Rendering data from models

Once the serialized data has been created with `serializeData()` on the model (NOPE, VIEW), it's passed to the `View`'s `render()` which looks for a template to render with the data. The template is set on the `View`'s `template` property (or set by the `getTemplate` method if you need to do it dynamically). Underscore templates by default. Easy to switch, though.

#### ItemView vs. CollectionView

You can extend from `Marionette.ItemView` to represent a single data object (can be a model or a collection) and render a template with it. You can also extend from `Marionette.CollectionView` to represent a collection, and it will iterate over the collection and render a view for each model (you can use the same view for every collection, or you can mix and match dynamically).

An ItemView might look like this:

```js
var TodoView= Marionette.ItemView.extend({
    tagName: 'li',
    template: '<span class="checkbox" {{checked}}></span><span class="text">{{ text}}</span><span class="delete"></span>',
    events: {
        'click .checkbox': 'toggleChecked',
        'click .delete' : 'deleteItem'
    },

    serializeData: function() {
        return {
            checked: this.model.get('checked') ? 'checked':'',
            text: this.model.get('text')
        };
    },

    toggleChecked: function() {
        //logic for checking the box
    },

    deleteItem : function () {
        //logic for deleting the todo
    }
});
```

...whereas a CollectionView might look like this:

```js
var TodoListView = Marionette.CollectionView.extend({
    childView: TodoView,
    tagName: 'ul'

});
```