# Sinatra notes

---

**Table of Contents:**

[toc]

---

## A basic application

A super simple Sinatra app looks like this:

```ruby
require 'sinatra'
 
# Add these two lines below if you're on the Learn IDE!
# set :bind, ENV["HOST_IP"]
# set :port, ENV["HOST_PORT"]
 
get '/' do
  "Hello, World!"
end
```

Those commented lines... figure out why that's necessary. I think it's just to fully show the IP address and port when doing it non-locally, so that you can navigate to it from the Learn IDE.

Run with `ruby app.rb`.

This is... uncommon. Usually a Sinatra app has a modular structure and is run using the `config.ru` Rack convention.

---

Ugh okay I’m finally annoyed with this enough to ask. Does anyone @here have the same problem I do (not the most recent update, it’s been like this forever for me) where the terminal in the Learn IDE wraps new lines onto existing lines? Like, if you type until it goes to the next line, it doesn;t 

So, this is something I’ve been meaning to ask: Does anyone @here have the same problem I do (not the most recent update, it’s been like this forever for me) where the terminal in the Learn IDE wraps new lines onto existing lines? Like, if you type until it goes to the next line, it doesn’t actually make a new line and start fresh from there, it just starts overwriting the current line from the beginning once it hits 80 columns!

---

## The application controller

The `app.rb` file is the application controller:

```ruby
require 'sinatra'

class App < Sinatra::Base

  get '/' do 
    "Hello, world!"
  end

end
```

Sinatra must be required, and your application inherits from the `Sinatra::Base` class. A Sinatra method there responds to `GET` requests for `/` with "Hello, world!".

## The Modular Sinatra Pattern

The previous stuff is bullshit.

### The `config.ru` file

Looks something like this:

```ruby
require 'sinatra'
 
require_relative './app.rb'
 
run Application
```

The `run` method is a "mounting" method. You mount a controller in the `config.ru` file with that last line (the controller being any Sinatra controller class).

### A Sinatra Controller

A Sinatra controller is basically just a class that inherits from `Sinatra::Base`. The `config.ru` requires it to `run`. See here:

```ruby
class Application < Sinatra::Base

  get "/" do
    "Hello, world!"
  end

end
```

The class name can be anything descriptive about that controller. No weird naming conventions required.

#### Defining routes

Route methods match their HTTP verbs (`get` for `GET`, `post` for `POST`, etc.) and are called with an argument (the path) and a block (the code to execute when the path is requested), the return value of which is added to the body of the response, if I'm not mistaken. Anyway, this:

```ruby
get '/medicines' do
  ...
end
```

...is the same as this:

```ruby
get('/medicines') { ... }
```

Obviously.

#### Named parameters in routes, etc.

See [here](http://www.sinatrarb.com/intro.html#Routes) for a bunch of fun stuff you can do with route names. Named parameters, optional parameters, wildcards, and a lot more.

#### Pass data into erb

You can pass data into an [erb template](erb.md) using `@instance_variables`. However, remember that instance variables do not persist between different route methods in an application. They only exist within that controller route and the erb template.

### Separation of Concerns—Making multiple controllers

If you have, like, CRUD actions on `/products` and on `/orders`, your controller will get messy quickly. You want to define multiple controllers for different models on your site, if that's the case. You might have `/products`, `/products/new`, `products/:id`, `products/:id/edit`, etc., and then also have `/orders`, `/orders/new`, `orders/:id`, `orders/:id/edit`, etc. You don't want that shit all in one controller! So, make multiple:

```ruby
class ProductsController < Sinatra::Base
 
  get '/products' do
    "Product Index"
  end
 
  get '/products/:id' do
    "Product #{params[:id]} Show"
  end
 
end
```

and

```ruby
class OrdersController < Sinatra::Base
 
  get '/orders' do
    "Order Index"
  end
 
  get '/orders/:id' do
    "Order #{params[:id]} Show"
  end
 
end
```

But, you can only run one controller at a time, so in `config.ru` you put:

```ruby
require 'sinatra'
 
require_relative 'app/controllers/products_controller'
require_relative 'app/controllers/orders_controller'

use ProductsController
run OrdersController
```

Note that you `use` one (or many) controller(s), and `run` the only one controller. In this example, it doesn't matter which is which. But in reality, you're using one as "Middleware" and running the other. It matters, but I'm not sure why yet, so I'm going to leave it for now.

## Shotgun development

You can use `shotgun` (gem) instead of `rackup` to make testing a little easier. Starting your application with `shotgun config.ru` means that your code will be reloaded every time a request is made, so you can edit on the fly and not have to keep restarting your server to test changes.

## File structure

Here is the basic file structure of a Sinatra application:

```
├── Gemfile
├── README.md
├── app
│   ├── controllers
│   │   └── application_controller.rb
│   ├── models
│   │   └── model.rb
│   └── views
│       └── index.erb
├── config
│   └── environment.rb
├── config.ru
├── public
│   └── stylesheets
└── spec
    ├── controllers
    ├── features
    ├── models
    └── spec_helper.rb
```

The `app/` directory is where we do most of our code.

The `application_controller.rb` file contains a controller class. It represents an instance of your application when the server is up and running. Sometimes, you might want the other controller classes to inherit from this main controller file (as opposed to just `Sinatra::Base`) to get a bunch of default behaviors.

Sinatra is configured by default to look for `.erb` files/views in the directory called `views`.

## Configuration

Configure block in the controller:

```ruby
configure do
  set :views, "app/views"
  set :public_dir, "public"
end
```

Tells the controller where to look for your views and your public resources.

## Sessions

Sessions are stored on a `session` hash and are sent to the user by way of a browser cookie. The browser sends the cookie along with every request, and (presumably, I might be wrong) the server sends it back to the browser to potentially update it, and the cycle starts over again. That way, persistence can be achieved.

### Enabling sessions

To enable sessions, do it in the controller in a `configure` block:

```ruby
configure do
  enable :sessions
  set :session_secret, "secret"
end
```

First, you enable the setting for sessions. Simple.

Then, you set an encryption key that will be used to set a `session_id`, which is an encrypted string associated with a user and stored as a browser cookie. The `"secret"` can be anything.

### Accessing data on `session`

The session is represented by a hash, called... `session`... and it looks a little like this:

```ruby
@session = {
  "session_id"=>  
    "dd32f512ee239ad74aa6f10c8cad37ce28d6c6922eff252ed641b1017130fe22", 
  "csrf"=> "040e9777d4dfae03bb1e6498f2a75482", 
  "tracking"=>{ 
    "HTTP_USER_AGENT"=> "e193e9e937caa9a19ca483f046281aae77d2216b", 
    "HTTP_ACCEPT_LANGUAGE"=> "66eae971492938c2dcc2fb1ddc8d7ec3196037da"
  }
}
```

You can access data on it the same way you'd think:

```ruby
session["session_id"]
```

And you can also assign data to it, to be able to access it in your views, for example:

```ruby
get "/hey" do
  session["name"] = "Vicky"
  @session = session
  erb :hey
end
```

### Flashing messages

#### Setup

To flash messages, we'll use the `rack-flash3` gem. Add it to your gemfile:

```ruby
gem 'rack-flash3'
```

Then require `rack-flash` (no "3") in your `environment.rb` file:

```ruby
...

require 'rack-flash'

...
require_all 'app'
require_all 'lib'
# etc.
```

You need to then enable sessions in your controller and use the `Rack::Flash` middleware:

```ruby
configure do
  enable :sessions
  set :session_secret, "secret"
  use Rack::Flash
end
```

#### Using flashed messages

Flash a message by setting it like a hash:

```ruby
post '/songs' do
  flash[:message] = "Successfully created song."
  redirect to("/songs/#{@song.slug}")
end
```

And use it in your view:

```html
<%= flash[:message] if flash.has?(:message) %>
```

## Method override (e.g. how to `PATCH`, `DELETE`, etc.)

Apparently you have to use some Rack middleware to be able to send/receive `PATCH` requests (or non-standard methods? Maybe? Not really sure). Put this in your `config.ru` file:

```ruby
use Rack::MethodOverride
run App # blah blah blah, etc. 
```

Then, if you want a form to send a `PATCH` request, for example, you'd put this in your view:

```html
<form action="/models/<%= @model.id %>" method="post">
    <input id="hidden" type="hidden" name="_method" value="patch">
    <input type="text" ...>
</form>
```

The `MethodOverride` middleware intercepts all requests and looks through each one to see if it includes `name="_method"`. If it does, it changes the type of the request to what the `value` attribute is set to. In this case, `value="patch"` so it becomes a `PATCH` request.

Same goes for `DELETE`.

## ActiveRecord and good CRUD design

See [activerecord.md](activerecord.md) for more specific notes on ActiveRecord. See [this Learn lesson](https://learn.co/tracks/full-stack-web-development/sinatra/activerecord/activerecord-in-sinatra) for a good overview of designing your application routes/controller behavior around CRUD management of records.

## RESTful URLs

General rule of thumb:

HTTP VERB | ROUTE | Action | Used For
--- | --- | --- | ---
GET | '/posts' | index action | index page to display all posts
GET | '/posts/:id' | show action | displays one blog post based on ID in the url
PATCH (Sinatra POST) | '/posts/:id/edit' | edit action | edits one blog post based on ID in the url
DELETE (Sinatra POST) | '/posts/:id/delete' | delete action | deletes one blog post based on ID in the url
POST | '/posts' | create action | creates one blog post

## Login/logout pattern

From [this lesson/lab](https://learn.co/tracks/full-stack-web-development/sinatra/activerecord/user-authentication-in-sinatra?batch_id=306&track_id=12615):

- Signing up for an app is nothing more than submitting a form, grabbing data from the `params` hash, and using it to create a new user.
- Logging in is nothing more than locating the correct user and setting the `:id` key in the `session` hash equal to their user ID.
- Logging out is accomplished by `#clear`ing all of the data from the `session` hash.

## Helper methods

You don't wanna put too much logic in your views/controller routes, so make a helper section in your controller class (dunno why it's in a block but oh well):

```ruby
class App < Sinatra::Base
  ...
  
  def helper do
    def logged_in?
	  !!session[:user_id]
    end

    def current_user
	  User.find(session[:user_id])
    end
  end
end
```