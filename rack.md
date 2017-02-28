# Rack notes

See [http.md](http.md) for more general HTTP notes.

## Basic Rack app

Just set up a class with a method `#call`:

```ruby
class Application
 
  def call(env)
    resp = Rack::Response.new
    resp.write "Hello, World"
    resp.finish
  end
 
end
```

The method `#call` creates a new `Rack::Response` object, writes "Hello, World" to the body of the response with `Rack::Response#write` (you can write as many times as you'd like before `finish`ing), then completes the response with the `Rack::Response#finish` method. The `env` passed to `#call` holds the request info, but we won't use it just yet. Rack sets our status codes and headers by default, so we won't worry about those either.

Then, you should have a `config.ru` file:

```ruby
require_relative './application.rb'

run Application.new
```

And that's it!

Now just run the server:

```
user@lappytoppy:~$ rackup config.ru
```

(Apparently in some environments you have to use `rackup config.ru -b 0.0.0.0` or `rackup config.ru --host 0.0.0.0`, check up on that)

Now, if you go to `http://localhost:3030/` or whatever port it says, you should see `Hello, World` on the page in your browser.

## The `Rack::Request` object

### The `env` parameter

A parameter is passed to `#call` (`env`) which contains the request info. You can parse this by passing it to a new `Rack::Request` object:

```ruby
class Application
 
  def call(env)
    resp = Rack::Response.new
    req = Rack::Request.new(env)
    
    resp.write "Hello, World"
    resp.finish
  end
 
end
```

### Methods

The `Request` object has a bunch of [useful methods](http://www.rubydoc.info/gems/rack/Rack/Request) on it.

### The `#path`

For example, you can get the path that was requested with `Rack::Request#path`. With that, you can filter your requests so you send responses tailored to the resource requested:

```ruby
class Application
 
  @@items = ["Apples","Carrots","Pears"]
 
  def call(env)
    resp = Rack::Response.new
    req = Rack::Request.new(env)
 
    if req.path.match(/items/)
      @@items.each do |item|
        resp.write "#{item}\n"
      end
    else
      resp.write "Path Not Found"
    end
 
    resp.finish
  end
end
```

### User input, like search parameters

You might search GitHub for apples. It'll send a `GET` request, and the URL will look like this: `https://github.com/search?q=apples`. The path is `search`, then there's a `?` followed by a key/value pair of `q=apples`. 

That section after the `?` is called the `GET` parameters. Rack gives them to you in a basic hash with `Rack::Request#params`. Awesome!

```ruby
class Application
 
  @@items = ["Apples","Carrots","Pears"]
 
  def call(env)
    resp = Rack::Response.new
    req = Rack::Request.new(env)
 
    if req.path.match(/items/)
      # ...
    elsif req.path.match(/search/)
 
      search_term = req.params["q"]
 
      if @@items.include?(search_term)
        resp.write "#{search_term} is one of our items"
      else
        resp.write "Couldn't find #{search_term}"
      end
 
    else
      resp.write "Path Not Found"
    end
 
    resp.finish
  end
end
```

### Setting status codes

Sometimes you gotta update status codes yourself. Rack does 'em mostly automatically, but if a page isn't found, you might want to send a 404:

```ruby
class Application
 
  def call(env)
    resp = Rack::Response.new
    req = Rack::Request.new(env)
 
    if req.path=="/songs"
      resp.write "You requested the songs"
    else
      resp.write "Route not found"
      resp.status = 404
    end
 
    resp.finish
  end
end
```

So just set it with `Rack::Response#status`. Info on status codes can be found [here](http.md#status).