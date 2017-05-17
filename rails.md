# Rails notes


---

**Table of Contents:**

[toc]

---

## The basics

[This](https://www.youtube.com/watch?v=KKQ8lpEyw2g) is a great (but long) intro to Rails and the basics of making a Rails app.

### Making a new project

Start a new rails project:

```
user@lappy:~$ rails new project-name
```

Project name should be lowercase and separated by dashes. You can also skip initializing a testing framework by passing the `-T` flag:

```
user@lappy:~$ rails new project-name -T
```

### Running a local server

Start the server:

```
user@lappy:~$ rails server
-- or --
user@lappy:~$ rails s
```

Voilà!

Any changes you make within the `app/` directory should be available with a refresh of the browser (no need to restart the server).

### Running a console to interface with the app

A little like `tux` for Sinatra:

```
user@lappy:~$ rails console
-- or --
user@lappy:~$ rails c
```

Nice.

### Workflow for creating an app

You're gonna want to do something along these lines, in this order:

1. Plan out the URLs - like a client-first API
2. Plan out the database structure for those initial features and create migrations
3. Do I have a model for that table? If no, generate the model(s)
4. Play with the model and database with `rails console` to make sure it works

## Generating stuff

For future reference:

```
user@lappy:~$ rails generate ...
-- or --
user@lappy:~$ rails g ...
```

Same thing, again.

### Scaffold & resource

You can generate scaffolds and resources, but you generally want to do everything with the most specific generator in mind.

- `rails g scaffold ...` - Generates controller, routes, model, migration, views

- `rails g resource ...` - Generates controller, routes, model, migration

### Model

You can generate a model and a migration (plus tests) at the same time like this:

```
$ rails g model Post title:string content:text
```

### Controller

Generate a controller (plus views, tests, helper, and assets like coffeescripts and scss) like this:

```
$ rails g controller Posts
```

You can also provide it with actions that you want templated:

```
$ rails g controller Posts index show edit
```

...which will make the following file:

```ruby
# app/controllers/posts_controller.rb

class PostsController < ApplicationController
  
  def index
  end
  
  def show
  end
  
  def edit
  end
  
end
```

...and it will also create the corresponding views for those actions, as well as the routes to those actions in `app/config/routes.rb`. Neat!

### Migrations

Make a migration like this:

```
$ rails generate migration CreatePosts title:string
```

makes:

```ruby
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.string :title
    end
  end
end
```

You can always edit this file later on, you're just initially generating it with some stuff filled in.

Then, you can `rake db:migrate` like a normie.

### What should you use?

Looks like generating the resource is a good standard action. It doesn't generate a lot of the unnecessary stuff that generating a scaffold or a controller would.

```bash
rails g resource Account name:string payment_status:string`
```

## Routing

A rails "route" matches a **URL** to a **controller** and an **action**.

### Drawing routes

Say I want to be able to go to `localhost:3000/about` and get a static html 'about' page. If I go there with a just-initiated application running, I'll get an error that says `No route matches [GET] "/about"`.

Go to `config/routes.rb` and add the route within the `draw` block:

```ruby
get '/about', to: 'static#about'
```

There's actually an easier way to do this, and I'm not sure of the differences between this and the two argument method I just stated (which can be phrased as `get("about", {to: "static#about"})` as well):

```ruby
get '/about' => 'static#about'
```

This has a couple pieces:

- `get` - the HTTP verb (associated with the route)
- `'/about'` - the path (in the URL, that this route maps to... not sure if preceding slash is necessary)
- `'static#about'` - the controller action (the method `#about` in the controller `StaticController`—an "action" is just another name for a controller method)

See [here](http://guides.rubyonrails.org/routing.html) for more.

From there, you're gonna have to make a `StaticController` class (inherit from `ApplicationController`) and define the `#about` method.

### Controller actions

So, make `app/controllers/static_controller.rb`, put this inside it:

```ruby
class StaticController < ApplicationController
  ...
end
```

Then add the controller action `#about`:

```ruby
class StaticController < ApplicationController

  def about
  end

end
```

Wonderful! There's our action. Let's see how it works...

<a name="rendering_views"></a>

### Rendering views

Now, if you refresh, you'll see that it's complaining about there not being a template at `static/about`. Seems that Rails assumes you'll be rendering a template at the end of your action that resides in `app/views/[controller name]/[method name].html.erb`. This is **implicit rendering**. You can also **explicitly render** a view file by creating a template:

```html
<!-- app/views/static/some_page.html.erb -->

<h1>Hello from some page</h1>
```

...and supplying it to the action:

```ruby
def about
  # you can use the full path:
  # render "static/some_page"
  
  # or, you can rely on the fact that it will look in the
  # views subdirectory which matches the controller name:
  render "some_page"
  
  # the latter is best practice
end
```

...and you will see:

<div style="box-shadow: 0 0 5px #aaa;"><div style="border: 1px solid gray; padding: 0.2em; background-color: #f5f2f0">🌎 <span style="font-size: 1em; background-color: white; border: 1px solid gray; padding: 0.2em 5em 0.2em 0.5em;">http://localhost:3000/about</span></div><div style="border: 1px solid black; border-top: none; padding: 1em 1em 3em 1em;">

<span style="font-size: 2em; font-weight: bold; font-family: serif;">    
  Hello from some page
</span>

<p style="font-family: serif;">
</p>

</div></div>

But, if you want to implicitly render, you can have a template:

```html
<!-- app/views/static/about.html.erb -->

<h1>Hello from the about page</h1>
```

...and leave the action render-free:

```ruby
def about
end
```

...and you will see:

<div style="box-shadow: 0 0 5px #aaa;"><div style="border: 1px solid gray; padding: 0.2em; background-color: #f5f2f0">🌎 <span style="font-size: 1em; background-color: white; border: 1px solid gray; padding: 0.2em 5em 0.2em 0.5em;">http://localhost:3000/about</span></div><div style="border: 1px solid black; border-top: none; padding: 1em 1em 3em 1em;">

<span style="font-size: 2em; font-weight: bold; font-family: serif;">    
  Hello from the about page
</span>

<p style="font-family: serif;">
</p>

</div></div>

### Resourceful routing

Check this shit out (taken from [here](http://guides.rubyonrails.org/routing.html)):

In Rails, a resourceful route provides a mapping between HTTP verbs and URLs to controller actions. By convention, each action also maps to a specific CRUD operation in a database. A single entry in `routes.rb`, such as:

```ruby
resources :photos
```

...creates seven different routes in your application, all mapping to `PhotosController`:

HTTP Verb | Path | Controller#Action | Used for
--- | --- | --- | ---
GET | /photos | photos#index | display a list of all photos
GET | /photos/new | photos#new | return an HTML form for creating a new photo
POST | /photos | photos#create | create a new photo
GET | /photos/:id | photos#show | display a specific photo
GET | /photos/:id/edit | photos#edit | return an HTML form for editing a photo
PATCH/PUT | /photos/:id | photos#update | update a specific photo
DELETE | /photos/:id | photos#destroy | delete a specific photo

That's _cray._ You can also make singular resources with `resource :photo`, which makes singular routes instead, as well as singular helpers. See [here](http://guides.rubyonrails.org/routing.html), again, for more explanation.

#### Path and URL Helpers

Creating a resourceful route will also expose a number of helpers to the controllers in your application. In the case of `resources :photos`:

- `photos_path` returns `"/photos"`
- `new_photo_path` returns `"/photos/new"`
- `edit_photo_path(:id)` returns `"/photos/:id/edit"` (for instance, `edit_photo_path(10)` returns `"/photos/10/edit"`)
- `photo_path(:id)` returns `"/photos/:id"` (for instance, `photo_path(10)` returns `"/photos/10"`)

Each of these helpers has a corresponding `[route name]_url` helper (such as `photos_url`) which returns the same path prefixed with the current host, port and path prefix.

Hell yeah. Singular resources also make singular helpers like these. See [here](http://guides.rubyonrails.org/routing.html), again, for more explanation.

#### Restricting actions on a resource route

You can restrict the actions so that the `resources` route drawn will only deal with certain ones:

```ruby
resources :posts, only: [:show, :index]
```

...or bar them from specific actions:

```ruby
resources :posts, except: :destroy
```

#### Scoping resources

This is very similar to namespacing resources, but I believe the difference is that scoped resources/URLs will still be grouped under the scope, but the controller actions and helper prefixes won't need the scope name (whereas the namespace would). This is a bad explanation. It's bad because I'm not quite sure of it. Anyway, check it out:

```ruby
scope '/admin' do
  resources :stats
end
```

Which will allow `GET /admin/stats` to route to `stats#index` (etc.) like normal and also have a normal helper prefix like `stats_path`. It's basically telling Rails that you want to use a URL prefix of `/admin` for the specified resources.

#### Namespacing resources

Scoping is nice, but what if we start handling a bunch of different controllers under the `admin` scope? It'll get a little hard to keep track of. Let's fix that by grouping those `admin`-related controllers together and namespacing them.

First let's make an `admin` directory in `app/controllers`, and move the `app/controllers/stats_controller.rb` file into `app/controllers/admin`. When you make a subdirectory in `controllers`, Rails makes an appropriately-named module to put those classes under. So you are expected to namespace the classes accordingly:

```ruby
# app/controllers/admin/stats_controller.rb

class Admin::StatsController < ApplicationController

  def index
    # ...
  end

end
```

When you have the controller in the module like this, Rails also expects views for the controller to be in a similarly-structured directory. Make a new `admin` directory (again) in `app/views/admin`, then make a `app/views/admin/stats` directory to place your view files (basically, for example, moving `app/views/stats/index.html.erb` to `app/views/admin/stats/index.html.erb`).

So, now that we have the structure all worked out, we can have a scoped route that references our new `admin` module:

```ruby
scope '/admin', module: :admin do
  resources :stats, only: [:index]
end
```

Basically what you've done here is told Rails that you want a URL prefix of `/admin`, and that all the routes will be handled by controllers in the `Admin` module (a.k.a. in the `admin` subdirectory we made and `Admin::...` module namespace we specified on the controller).

If you know you're gonna want to use the same name ("admin") for both the URL prefix and the module name, we can shorten this syntax using `namespace` instead of `scope ..., module: ...`

```ruby
namespace :admin do
  resources :stats
end
```

**_NOTE:_** There is a difference between `namespace` and `scope, module` that you have to be aware of. If you use `namespace`, your helper prefixes will, themselves, be prefixed with `admin`, too. For example, the methods will look like `admin_stats_path` instead of `stats_path`.

See [here](http://guides.rubyonrails.org/routing.html), again, for more explanation.

#### Nesting resources

Look [here](http://guides.rubyonrails.org/routing.html) for nesting info. It's good. I don't wanna type it. [This lesson](https://github.com/learn-co-students/routing-nested-resources-reading-v-000) is also really good for nested resources.

Basically if you do this:

```ruby
# config/routes.rb

resources :authors do
  resources :posts
end
```

...then you get nested resources where `posts` stuff is accessible "through" `authors`. A.K.A. you'll have a url like `/authors/3/posts/1/edit`, etc. You'll also want to access, say, `authors/3/posts` with a helper like `author_posts_path(some_author)`. You get the picture.

## Callback macros

### `#before_action`

You can use the `#before_action` macro like this, to execute a bit of code before an action happens:

```ruby
class ArticleController < ApplicationController
  
  before_action :set_article, only: [:show, :edit, :update, :destroy]
  
  ...
  
  private
  
    def set_article
      @article = Article.find(params[:id])
    end
    
end
```

That way, `#set_article` gets called before eack of the associated actions (`#show`, `#edit`, `#update`, `#destroy`), and sets the `@article` instance variable with the `:id` from the URL of the request.

## Validation

The `crud-with-validations-lab-v-000` lab has some good examples of validation and custom validation on a model. Worth going back and skimming. Also, check out [the Rails guide on validations](http://guides.rubyonrails.org/active_record_validations.html), which is great.

### Validate stuff

You can validate certain things on a model, to make sure they are present/fit some criteria before the object can be saved to the database, by using `validate :attribute, options_hash`. You can then call `#valid?` on a newly created object, and it will return whether it is valid or not. Using `#save` or `create` will return true or false based on whether it validated, or they can raise an exception if you use a `!` on the end:

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true
end
 
p = Person.new
p.valid? #=> false
p.save #=> false
p.save! #=> EXCEPTION
```

#### Length is a good one

Check it out:

```ruby
class Person < ActiveRecord::Base
  validates :name, length: { minimum: 2 }
  validates :bio, length: { maximum: 500 }
  validates :password, length: { in: 6..20 }
  validates :registration_number, length: { is: 6 }
end
```

#### Also, uniqueness!

Check it, yo:

```ruby
class Account < ActiveRecord::Base
  validates :email, uniqueness: true
end
```

Now two accounts cannot be created with the same email address. Introspect on the error messages instead of making your own, and you've become a BOSS.

#### Also, inclusion in a list!

Check to see if an attribute is a member of a list:

```ruby
class Post < ActiveRecord::Base
  validates :category, inclusion: { in: ["Fiction", "Non-Fiction"] }
end
```

#### How 'bout some formatting?

```ruby
class Person < ActiveRecord::Base
  validates :name, format: { without: /[0-9]/, message: "does not allow numbers" }
  validates :email, uniqueness: true
end
```

yo yo yo yo yo, cool, cool cool cool

#### Custom messages

You can make custom messages, too:

```ruby
class Person < ActiveRecord::Base
  validates :not_a_robot, acceptance: true, message: "Humans only!"
end
```

#### Custom validators

##### From Learn

_I'm copy/pasting this straight from [this Learn lesson](https://learn.co/tracks/full-stack-web-development/rails/validations-and-forms/activerecord-validations), no shame._

There are three ways to implement custom validators, with examples in [Section 6 of the Rails Guide](http://guides.rubyonrails.org/active_record_validations.html#performing-custom-validations).

Of the three, `#validate` is the simplest, because all you need to do is define an instance method that is invoked by `#validate`. This is probably the best way to start with most custom validations, because everything is in one place, and you can come back later to re-organize if it starts to get more
complex.

- Calling `#validate` (that's "validate" without an "s") with the name of an instance method
	- Use this approach when you're not sure which to use. If you end up needing to use the same validation logic on a different model, you can easily extract the instance method into one of the ActiveModel classes and use `#validates` or `#validates_with` instead.
- Subclassing `ActiveModel::EachValidator` and invoking with an inflected key in the options hash
	- This is best for validating a single attribute on one model, especially one that you're already using built-in validators for.
	- For example, in the Rails Guide, they define `EmailValidator` and then pass the `email: true` key-value pair to `#validates` to invoke it.
- Subclassing `ActiveModel::Validator` and invoking with `#validates_with`
	- This approach is best when you want to do a whole bunch of validations on several different models. You can just call `validates_with MyValidator` on each of them.

So, to recap:

- `validate` for quick custom validations that you can extract later.
- `EachValidator` and `validates` for validating one specific attribute.
- `Validator` and `validates_with` for doing many validations in one pass.

##### The `#validate` (no 's') method

This is a basic custom validation using `#validate`:

```ruby
class Post < ActiveRecord::Base
  validate :clickbait_title

  def clickbait_title
    if self.title && !self.title.match(/(Won't Believe|Secret|Top \d+|Guess)/)
      self.errors.add(:title, "must be clickbait-y")
    end
  end
end
```

### Check errors

An object created with `.new` will not have errors or have a "valid" boolean until you have called a method with the proper callback associated with it (`#valid?` counts as one of those methods).

Anyway, you can check the errors with `#errors` after validation. You can use `#errors#messages` to get a hash of attribute-related errors. You can also use `#errors#full_messages` to get sentences out of them:

```ruby
p = Person.new
p.errors.messages #=> empty
p.valid? #=> false
p.errors.messages #=> name: ["can't be blank"]
p.errors.messages[:name] #=> ["can't be blank"]
p.errors.full_messages #=> ["Name can't be blank"]
```

#### Displaying errors

You can use errors like flash messages if you just `render :view_name` when the object fails to validate, passing the object in as an instance variable and then doing the obvious:

```html
<% if @person.errors.any? %>
  <div id="error_explanation">
    <h2>There were some errors:</h2>
    <ul>
      <% @person.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```

You can also do this cool trick, adding a class to a field that makes it display some custom way if there are errors for it:

```html
<div class="field<%= ' field_with_errors' if @person.errors[:name].any? %>">
  <%= label_tag "name", "Name" %>
  <%= text_field_tag "name", @person.name %>
</div>
```

So, maybe a `div` with a class of `field_with_errors` has a `background-color: red;` style or something.

## Authentication

Maybe this shouldn't be its own section? Maybe it should?

### Strong params (restricting `params` for mass assignment)

You can use `#require` and `#permit` on the `params` hash object to restrict the values passed to a model method (or whatever). Check it out:

```ruby
# params is currently:
# {
#   user: {
#     username: "test_user",
#     email: "test@gmail.com",
#     password_digest: "45lbjk90csedr3kbj3nl2n8v9n4dt63m"
#   },
#   auth: {
#     other_stuff: "blah blah blah"
#   }
# }

params.require(:user)
# => returns:
# {
#   username: "test_user",
#   email: "test@gmail.com",
#   password_digest: "45lbjk90csedr3kbj3nl2n8v9n4dt63m"
# }
# ...and if :user isn't there, it raises an error

params.require(:user).permit(:username, :email)
# => returns:
# {
#   username: "test_user",
#   email: "test@gmail.com"
# }
# ...and that object, if you were to call `#permitted?` on
# it, would return true, whereas the original object would
# return false.
```

This prevents malicious requests from editing fields you didn't want to be editable, for instance.

It's a good idea to abstract that stuff, too, so you can do something like this:

```ruby
# app/controllers/posts_controller.rb
 
def create
  @post = Post.new(post_params)
  @post.save
  redirect_to post_path(@post)
end
 
def update
  @post = Post.find(params[:id])
  @post.update(post_params)
  redirect_to post_path(@post)
end
 
private
 
def post_params
  params.require(:post).permit(:title, :description)
end
```

But that can be done one better:

```ruby
# app/controllers/posts_controller.rb
 
def create
  @post = Post.new(post_params(:title, :description))
  @post.save
  redirect_to post_path(@post)
end
 
def update
  @post = Post.find(params[:id])
  @post.update(post_params(:title))
  redirect_to post_path(@post)
end
 
private
 
def post_params(*args)
  params.require(:post).permit(*args)
end
```

So, that way, `post_params` is kept pretty well-abstracted, but still has the option of specifying certain attributes to permit.

#### Allowing arrays in strong params

You have to [explicitly map a parameter to an empty array](http://stackoverflow.com/questions/16549382/how-to-permit-an-array-with-strong-parameters) if you want to be able to pass an array in your strong params.

For example:

```ruby

# If params looks like this:
# {
#    "utf8"=>"✓",
#    "post"=>{
#        "name"=>"Post title",
#        "content"=>"post content",
#        "tag_ids"=>["1", "2", ""]
#    },
#    "commit"=>"Create Post",
#    "controller"=>"posts",
#    "action"=>"create"
# }

params.require(:post).permit(:name, :content)
# => {
#        "name"=>"Post title",
#        "content"=>"post content"
#    }

params.require(:post).permit(:name, :content, :tag_ids)
# => {
#        "name"=>"Post title",
#        "content"=>"post content"
#    }

params.require(:post).permit(:name, :content, tag_ids: [])
# => {
#        "name"=>"Post title",
#        "content"=>"post content",
#        "tag_ids"=>["1", "2", ""]
#    }
```

### Login and user stuff

So, like you know, all it really takes to log in a user is storing the `user_id` on the `session` hash.

#### "Return guard" and requiring a user to be logged in

Check it, yo:

```ruby
def show
  return head(:forbidden) unless session.include? :user_id
  @document = Document.find(params[:id])
end
```

Nice. Pretty self-explanatory, I think. The `head` method is new. Returns the HTTP status code `403 Forbidden` with `head(:forbidden)`.

But, you're probably gonna be using this line in multiple controller actions, so lets abstract it and throw it in a `before_action`:

```ruby
class DocumentsController < ApplicationController
  before_action :require_login
 
  def show
    @document = Document.find(params[:id])
  end
 
  def index
  end
 
  def create
    @document = Document.create(author_id: user_id)
  end
 
  private
 
  def require_login
    return head(:forbidden) unless session.include? :user_id    
  end
end
```

Awesome! If you want to restrict that `before_action` to only a few actions, you could do:

```ruby
...
  before_action :require_login, only: [:show, :create]
...
```

That would allow any user to see the `index`. If you don't want to explicitly tell it which ones to _do_ the `before_action` (e.g. you have a lot of actions and you keep adding more, but there are only a few actions you'd like to skip, like `index`...), you can do this:

```ruby
...
  before_action :require_login
  skip_before_action :require_login, only: [:index]
...
```

I feel like you can do that with `..., except: [:index]` instead, but whatevs. These are called "filters". You can read more about them [here](http://guides.rubyonrails.org/action_controller_overview.html#filters).

### Secure passwords

You can use the `has_secure_password` macro, like in Sinatra, to make sure a model has a hashed password. The macro allows you to do nice things like `user.authenticate(params[:password])`.

It also gives us a few neat `before_save` features—it will compare a `:password` and a `:password_confirmation` to make sure they match if you are creating a `User` with strong params with both of those keys in the hash, for example. Check it out:

```html
<!-- app/views/user/new.html.erb -->
<%= form_for :user, url: '/users' do |f| %>
  Username: <%= f.text_field :username %>
  Password: <%= f.password_field :password %>
  Password Confirmation: <%= f.password_field :password_confirmation %>
  <%= f.submit "Submit" %>
<% end %>
```

```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  has_secure_password
end
```

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def create
    user = User.new(user_params).save
  end
 
  private
 
  def user_params
    params.require(:user).permit(:name, :email, :password, :password_confirmation)
  end
end
```

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def create
    @user = User.find_by(username: params[:username])
    return head(:forbidden) unless @user.authenticate(params[:password])
    session[:user_id] = @user.id
  end
end
```

In the `UsersController`, the `save` will throw an error if `user_params[:password]` and `user_params[:password_confirmation]` do not match.

### Logging out

Basically just clear the session. You can do `reset_session` or `session.clear`... or you can do `session.delete(:user_id)` if you'd like to be more specific. You should look up `reset_session`, I've never actually used it at time of writing.

### Omniauth and the OAuth protocol

Add `gem 'omniauth'` and `gem 'omniauth-facebook'` (or google, or whatever) to your Gemfile. Then, create a file at `config/initializers/omniauth.rb` with this middleware:

```ruby
# config/initializers/omniauth.rb

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :facebook, ENV['FACEBOOK_KEY'], ENV['FACEBOOK_SECRET']
end
```

That way you can pull from environment variables that you can set and not have it stored publicly on GitHub or whatever. Basically, you're gonna register your app with the provider you choose (facebook, google, etc.) and then they'll give you instructions on how to set it up.

For facebook, follow their instructions to create an app, then grab the App ID and the Secret Key and set them as environment variables on your system:

```bash
export FACEBOOK_KEY=<app_key>
export FACEBOOK_SECRET=<secret_key>
```

You're also going to need to remember to do "Add Platform" > "Website", set the site URL and App Domain respectively to http://localhost:3000 and localhost.

Then you should be able to throw a link in one of your views:

```html
<!-- app/views/static/home.html.erb -->
 
<%= link_to("login with facebook!", "/auth/facebook") %>
```

_When you export those environment variables, either make a new terminal session to start the server or start it in the same terminal session, otherwise they won't be recognized, obviously. I forgot about this and tried to start the server in an existing (but separate) terminal session and couldn't figure out why it didn't recognize the keys._

When you click that link, you'll be redirected to facebook, where you'll log in, and then facebook will redirect you (again) to your site's `/auth/facebook/callback` URL (or whatever provider). You don't have that set up yet, so make a route like this:

```ruby
# config/routes.rb

...
  get '/auth/facebook/callback', to: 'sessions#create'
...
```

(or whatever controller action you're gonna use) and make the appropriate action in your controller.

Facebook is gonna send us back a hash like this:

```ruby
{
  "provider": "facebook",
  "uid": "10154316774896935",
  "info": {
    "name": "Keegan Leitz",
    "image": "http://graph.facebook.com/10154316774896935/picture"
  },
  "credentials": {
    "token": "EAADKXa1MIiIBAMtOOfpoIYjN4qY9JXZAG3POabc01moRV9UZBYQuU87fxgZCNRLIkdAabbL1JTzBe5zH5vfIF64zfV1PhFqZBrgWAZBiJtUbZCXTL6rRRA96oFrug2MVNLeOAlpqnCZCadX6GZAD8AmFwQPVFDMGQB0ZD",
    "expires_at": 1493570206,
    "expires": true
  },
  "extra": {
    "raw_info": {
      "name": "Keegan Leitz",
      "id": "10154316774896935"
    }
  }
}
```

Omniauth will parse this and throw it in the "request environment", on `request.env['omniauth.auth']`. Abstract it with an `auth` method, and use that to construct a user (`User` model/migration code omitted, but you can imagine how it would work... one note: `uid` might be out of range for a simple integer, so you can make a 64-bit int with `t.integer :uid, limit: 8` in your migration):

```ruby
# app/controllers/sessions_controller.rb
 
class SessionsController < ApplicationController
 
  def create
    user = User.find_or_create_by(:uid => auth['uid']) do |u|
      u.name = auth['info']['name']
      u.email = auth['info']['email']
    end
    session[:user_id] = user.id
  end
 
  def auth
    request.env['omniauth.auth']
  end
end
```

### Devise for authentication

You can use Devise to send confirmation emails, lock user accounts after a certain number of failed login attempts, and send password resets. You can also use it to allow multiple models to be signed in, each with different roles, granting different permissions.

See more about Devise in [the Devise notes](devise.md).

## Authorization

Authorization is distinct from Authentication in that it governs what a user can _do_, not whether the user is who they say they are. [This is a fantastic video](https://www.youtube.com/watch?v=wsbfUc-CPbg) of Avi going through Authorization patterns and metaprogramming.

CanCanCan is a gem that provides a lot of this functionality. See more about CanCanCan in [the CanCanCan notes](cancancan.md).

### Using `enum`

Yo, check it [out](http://edgeapi.rubyonrails.org/classes/ActiveRecord/Enum.html). That's AWESOME. Very good explanation at the top of the docs on that one. You can easily see how `enum` might be useful for keeping track of users' roles (as guest, normal user, administrator, etc.). For example, you might do this:

```ruby
class User < ActiveRecord::Base
  enum role: [:normal, :moderator, :admin]
end
```

..and then you can do this:

```ruby
user.admin?
```

...which, because of `enum`, it interprets as:

```ruby
user.role == 2
```

So cool. The role `enum` is stored as an integer in the database (do I have to add it as a column? **edit: yes**).

#### Checking if a user is a guest

You might have a `User` object which has not been persisted to the database (maybe not logged in, maybe filled out `/sign_up` but not yet saved/validated). That user is a "guest". You can check this with `#persisted?` like so:

```ruby
class User < ActiveRecord::Base
  enum role: [:normal, :moderator, :admin]
  
  def guest?
    persisted?
  end
end
```

Just a note: `user.role` will never return `'guest'` like this, since it's not a "real" role.

#### Using the methods in a common pattern

You can use these methods like this:

```ruby
class PostsController < ApplicationController
  def update
    @post = Post.find(params[:id])
    return head(:forbidden) unless current_user.admin? ||
                       current_user.moderator? ||
                   current_user.try(:id) == @post.id
    @post.update(post_params)
  end
  # more down here
end
```

Sweet. If you're using CanCanCan (see the [CanCanCan notes](cancancan.md)) you can use these roles to determine what abilities a user has:

```ruby
class Ability
  include CanCan::ability
 
  def initialize(user)
    user.can :read, Post
    return if user.guest?
 
    user.can :update, Post, {owner_id: user.id}
    return if user.normal?
 
    user.can :update, Post
    return if user.moderator?
 
    user.can :manage, Post if user.admin?
  end
end
```

Very cool. I don't _love_ that cascading pattern, but oh well.

## Views

### Files and structure

- We make views in the views directory...
	- `app/views`
- in a subdirectory named for the controller they are associated with...
	- `app/views/posts`
- named for the action associated with them...
	- `app/views/posts/index`
- with an extension of ".html.erb"
	- `app/views/posts/index.html.erb`

### Rendering

See [this section](#rendering_views). Also, see the [ActionView section](#actionview).

### Layouts

Default layout is assumed to be at `app/views/layouts/application.html.erb`. Rails will first look for layouts specific to your controller, though. So, if you have a `UsersController` it'll look first for `layouts/users.html.erb`. You probably only need the one default layout.

You can override this stuff pretty easily. Say you have a `ShoppingCartController` and you want to use `layouts/products.html.erb`, you can do this to set the default layout for the controller:

```ruby
class ShoppingCartController < ApplicationController
  layout "products"
end
```

...or, do this to specify a layout for an action:

```ruby
class ShoppingCartController < ApplicationController
  
  def list
    render layout: "products"
  end

end
```

...or, if you don't want to render a layout:

```ruby
class ShoppingCartController < ApplicationController
  
  def list
    render layout: false
  end

end
```

### Partials (e.g. `_form.html.erb`)

You can use "partials" to keep your templates DRY.

#### The basics

Say you have this view:

```html
<!-- app/views/posts/new.html.erb -->
 
<%= form_tag posts_path do %>
  <label>Post title:</label><br>
  <%= text_field_tag :title %><br>
 
  <label>Post Description</label><br>
  <%= text_area_tag :description %><br>
 
  <%= submit_tag "Submit Post" %>
<% end %>
```

...and this view:

```html
<!-- app/views/posts/edit.html.erb -->
 
<h3>Post Form</h3>
 
<%= form_tag post_path(@post), method: "put" do %>
  <label>Post title:</label><br>
  <%= text_field_tag :title %><br>
 
  <label>Post Description</label><br>
  <%= text_area_tag :description %><br>
 
  <%= submit_tag "Submit Post" %>
<% end %>
```

Aside from the first line of each form, these are basically exactly the same. So, let's make a partial for that shared code:

```html
<!-- app/views/posts/_form.html.erb -->

<label>Post title:</label><br>
<%= text_field_tag :title %><br>
 
<label>Post Description</label><br>
<%= text_area_tag :description %><br>
 
<%= submit_tag "Submit Post" %>
```

Notice that the partial is named with an initial underscore (`_form.html.erb`). Now, go back to the `new` and `edit` views, and use a `render "form"` to render the partial at the point where you need it:

```html
<!-- app/views/posts/new.html.erb -->
 
<%= form_tag posts_path do %>
 <%= render 'form' %>
<% end %>
```

...and:

```html
<!-- app/views/posts/edit.html.erb -->
 
<h3>Post Form</h3>
 
<%= form_tag post_path(@post), method: "put" do %>
  <%= render 'form' %>
<% end %>
```

Nice!

Keep in mind: 

- You wanna keep the `<% end %>` tag with the opening of the block (we didn't include it in the partial, see?)
- The partial can be named anything, as long as it starts with an underscore. When the partial is referenced, it's gotta be _without_ an underscore.
- The partial is assumed to reside in the same folder as the view, but you can specify a different folder like `render "posts/form"` if you want (no need to do `render "../posts/form"` apparently, because it will check from the base `views` folder if you specify a folder in the string, I guess)

#### Locals for partials

If we specify a different folder for a partial, such as with `render "authors/author"` in a `post` view, we might have to pass in an instance variable `@author` that `authors/_author.html.erb` depends on, and then maybe when we decide later that we don't want that partial to be rendered in the `post` view, we have a dangling `@author` variable in the controller that we have to clean up. That's a small example of a headache caused by dependencies for a partial.

We can avoid this by being explicit about what is passed to the partial, and create local variables to pass in instead:

```html
Information About the Post
<%= render partial: "authors/author", locals: {post_author: @post.author} %>
<%= @post.title %>
<%= @post.content %>
```

You use that nested `locals` hash to pass in local variables, where the key is the name of the variable and the value is what you want it to refer to.

So, where the original partial might have been:

```html
<ul>
  <li> <%= @author.name %></li>
  <li> <%= @author.hometown %></li>
</ul>
```

...it is now:

```html
<ul>
  <li> <%= post_author.name %></li>
  <li> <%= post_author.hometown %></li>
</ul>
```

You can also use a shorter syntax than explicitly speciying the `:partial` and `:locals` keywords:

```html
Information About the Post
<%= render "authors/author", post_author: @post.author %>
<%= @post.title %>
<%= @post.content %>
```

Nice.

#### Collections of partials

Say you want to render a partial for each element of a collection. You'd probably do something like this:

```html
<% @posts.each do |post| %>
  <%= render partial: "post", locals: {post: post} %>
<% end %>
```

Instead, you can do this:

```html
<%= render partial: "post", collection: @posts %>
```

That does the same thing! Cool, huh? You can do it with an even shorter syntax, too:

```html
<%= render @posts %>
```

Magic. This assumes that you will have a partial with the name of the model in the collection passed to it. You can actually even have a heterogenous collection like [customer, order, customer] and it will call the correctly named partial for each one! Whaaat!

If the collection is empty, the render method will return `nil`, so you will want to handle that. You can do so by passing it a default value with the `||` operator:

```html
<%= render(@posts) || "There are no blog posts!" %>
```

You need the `()` for precedence or whatever.

Also, if you want, you can also specify what bare word to use as the passed-in local for each element of the collection:

```html
<%= render partial: "post", collection: @posts, as: :post %>
```

Coolio. If you don't do this, it will use the name of the **partial** as the name of the local variable. That's pretty cool, too.

## Useful helper methods

### Formatting

```ruby
pluralize(5, "laptop")
# => "5 laptops"

pluralize(1, "laptop")
# => "1 laptop"
```

See [here](http://api.rubyonrails.org/classes/ActiveSupport/Inflector.html) for a ton more awesome formatting helper methods.

### Helper modules

Define helper methods in the appropriately named file in the `app/helpers` folder, namespaced under `PostHelper` or whatever. These methods will be available to your views the same way other ActionView stuff is (so you can use them like you would use `link_to`, `form_for`, etc.).

<a name="actionview"></a>

## ActionView and its methods

ActionView allows us to write HTML a lot more easily. The following is assumed to be implemented in the views, unless otherwise stated. Also see the [forms section](#forms).

### Linking

You can make a link like this:

```html
<%= link_to "About Page", "/about" %>
```

...which generates the following HTML:

```html
<a href="/about">About Page</a>
```

So, first argument is the inner HTML, second argument is the `href`.

#### Accessing a route name with `#[name]_path`

However, you don't really want to hard-code the path as `"/about"`. If you change the name of the action or URL or whatever, you're gonna have to go back and change that in the view manually. SCREW THAT.

Lemme show you something. Look at the route info with `rake routes`, then go into the rails console, and you can see the path for the "about" route (that route is named now! neat.) automatically:

```bash
user@lappy:~$ rake routes
     Prefix Verb URI Pattern            Controller#Action
      about GET  /about(.:format)       static#about
hello_world GET  /hello_world(.:format) static#hello_world
user@lappy:~$ rails c
001:0 > app.about_path
=> "/about"
002:0 > app.hello_world_path
=> "/hello_world"
```

You can use that `about_path` method directly in your view like this:

```html
<%= link_to "About Page", about_path %>
```

Now, if you want to change the route URL in `routes.rb` you can do so like this:

```ruby
# previously:
get "/about" => "static#about"

# change the URL but keep it named "about" so you
# can use the #about_path method:
get "/about_us_and_stuff" => "static#about", as: :about
```

Rails uses the URL to automatically name the route, but you can set a name for the route with `, as: :blah`. This way, when you are making links, you can always link to a **route** instead of to a static URL, taking future URL change into account.

#### Dealing with route variables

Something to consider: if you have a route variable, the `#[route name]_path` method accepts an argument that corresponds to that route variable. I believe you can pass multiple arguments for multiple route variables, too, but you'll have to try that out. So, say your route looks like this:

```ruby
get "/posts/:id" => "posts#show", as: post
```

...you will get an error if you try to check that path:

```bash
$ rails c
001:0 > app.post_path
=> # an error saying no route matches, missing
   # required keys: [:id], etc.
```

So you can pass it an argument to supply that route variable:

```bash
001:0 > app.post_path(1)
=> "/posts/1"
```

So you can do something like this in the view:

```html
<%= link_to "See Post", post_path(@post.id) %>
```

...or whatever.

#### Making things easier

Turns out you can also do this:

```html
<!-- Original -->
<%= link_to "See Post", post_path(@post.id) %>

<!-- Abstraction -->
<%= link_to "See Post", post_path(@post) %>
```

Rails will automatically call the `#id` method on that object. I _think_ `#link_to` calls the method by the named route variable (`:id` in the `/posts/:id` route URL) on the object passed in, or just uses what's passed in if that method doesn't exist.

**BUT WAIT!**

**THERE'S _MORE!_**

```html
<%= link_to "See Post", @post %>
```

WhaaAAaaaAAaaaAAat?

Yeah, you can do that, too. It implicitly calls `#post_path` based on the class of the object passed in (`@post` –> `Post`). But Avi mentioned that he tends to write his links at the previous level of abstraction.

#### DESTROYing with links! (a.k.a. specifying a method for a link, and asking for confirmation)

Check this fancy shit out:

```html
<% @people.each do |person| %>
<div class="person">
  <span><%= person.name %></span>
  <%= link_to "Delete", person, method: :delete, data: { confirm: "Really?" } %>
</div>
<% end %>
```

That makes the following link:

```html
<a data-confirm="Really?" rel="nofollow" data-method="delete" href="/people/1">Delete</a>
```

Uses a bit of JavaScript (provided by Rails) under the hood. Defaults to `GET` instead of `DELETE` if the user has JavaScript disabled. Pops up with a confirmation box before the link is followed. Nice.

## Testing with RSpec

Taken from [this lesson](https://learn.co/tracks/full-stack-web-development/rails/rails-models-basics/activerecord-models-and-rails).

So, if we want to create tests with RSpec right from the get-go:

```
$ rails new rails-activerecord-models-and-rails-readme -T
```

The -T flag tells the Rails project generator not to include TestUnit, the default testing framework.

Modify the gemfile by adding the following into the `:development, :test` group:

```ruby
gem 'rspec-rails', '~> 3.0'
```

Then:

```
$ bundle install
```

Then, create the initial RSpec config:

```
$ rails g rspec:install
```

Now, let's generate a model (and its associated migration and spec file, all in one go!):

```
$ rails g model post title:string description:text
```

You'll see a template spec file for the model in `spec/models/post_spec.rb`. Let's make some tests!

Your current spec file should look like this:

```ruby
require 'rails_helper'

RSpec.describe Post, type: :model do
  pending "add some examples to (or delete) #{__FILE__}"
end
```

Let's change it to this:

```ruby
require 'rails_helper'

describe Post do
  it 'can be created' do
    post = Post.create!(title: "My title", description: "The post description")
    expect(post).to be_valid
  end
end
```

Now, if you run your tests with `bundle exec rspec`, it'll throw an error because the database hasn't been migrated. Migrate the database with `rake db:migrate` and run the tests again.

### MVC testing

Some good examples of testing models, controllers, views, and features are in the `crud-with-validations-lab-v-000` lab. You may get more help out of the [Capybara notes](capybara.md), the [RSpec notes](rspec.md), or the [general "tests" notes](tests.md).

## Making forms

<a name="#csrf"></a>

### Cross-Site Request Forgery

Or, **CSRF**. You open your banking site and log in. You open a new tab and go to some sketchy site. The site runs some scripts, hijacks the session from the banking tab, and submits a form request to transfer money to their account. The bank doesn't know it's not you, so they follow through.

Rails disallows this from happening by requiring a unique authenticity token to be submitted with each form. The token is stored in the session, and can't be accessed outside of it. It'll throw an error if the authenticity token doesn't match.

You can place this token in a form like so:

```html
<form action="<%= posts_path %>" method="POST">
  <label>Post title:</label><br>
  <input type="text" id="post_title" name="post[title]"><br>
 
  <label>Post description:</label><br>
  <textarea id="post_description" name="post[description]"></textarea><br>
 
  <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

  <input type="submit" value="Submit Post">
</form>
```

But this is ugly and a terrible way to do forms with Rails.

<a name="forms"></a>

### Making better forms

#### The `#form_tag` method

We'll start with the form shown in the [CSRF section](#csrf). Let's use ActionView methods to help build out the form instead. The `#form_tag` method creates a form like this:

```ruby
form_tag some_path_to_post_to do
  ...
end
```

So:

```html
<%= form_tag posts_path do %>
  <label>Post title:</label><br>
  <input type="text" id="post_title" name="post[title]"><br>
 
  <label>Post description:</label><br>
  <textarea id="post_description" name="post[description]"></textarea><br>
 
  <input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>">

  <input type="submit" value="Submit Post">
<% end %>
```

#### The `#hidden_field_tag` method

We can make a hidden field for the authenticity token using `#hidden_field_tag` like this:

```ruby
hidden_field_tag :name_attribute, "value_attribute"
```

So:

```html
<%= form_tag posts_path do %>
  <label>Post title:</label><br>
  <input type="text" id="post_title" name="post[title]"><br>
 
  <label>Post description:</label><br>
  <textarea id="post_description" name="post[description]"></textarea><br>

  <%= hidden_field_tag :authenticity_token, form_authenticity_token %>

  <input type="submit" value="Submit Post">
<% end %>
```

#### The `#form_tag` method inserts the authenticity token automatically

But if you look at the rendered HTML, you'll see that just using `#form_tag` inserts the hidden authenticity token field automatically, so it's redundant. Now we have this:

```html
<%= form_tag posts_path do %>
  <label>Post title:</label><br>
  <input type="text" id="post_title" name="post[title]"><br>

  <label>Post description:</label><br>
  <textarea id="post_description" name="post[description]"></textarea><br>

  <input type="submit" value="Submit Post">
<% end %>
```

If you would like to change the method by which the form is submitted (like `patch` or `put`, instead of the default `post`):

```html
<%= form_tag posts_path, method: "put" do %>
  ...
<% end %>
```

#### The `#text_field_tag`, `#text_area_tag`, and `#submit_tag` methods

Seems like Rails (ActionView) has a lot of nice methods for constructing HTML! We can invoke the former two methods with the `name` attribute as a symbol, and the value of the latter with a string:

```html
<%= form_tag posts_path do %>
  <label>Post title:</label><br>
  <%= text_field_tag :'post[title]' %><br>

  <label>Post description:</label><br>
  <%= text_area_tag :'post[description]' %><br>

  <%= submit_tag "Submit Post" %>
<% end %>
```

You can also put the optional `value` attribute in the tags like this:

```html
...
  <%= text_field_tag :'post[title]', "default input text" %><br>
...
  <%= text_area_tag :'post[description]', "default textarea text" %><br>
...
```

The methods also add the `id` attributes as underscore-separated words derived from the name. Check out the final rendered form from the above template:

```html
<form action="/posts" accept-charset="UTF-8" method="post">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input type="hidden" name="authenticity_token" value="vq9SMVNk0CjwgZmYomFRhwbo5dfu7tI/2FiR7jOtlVgbj8r/zOO5oL+arU9N4PMm7WqxbUbXg4wqneW02ZfpMw==" />
  
  <label>Post title:</label><br>
  <input type="text" name="post[title]" id="post_title" /><br>
 
  <label>Post description:</label><br>
  <textarea name="post[description]" id="post_description"></textarea><br>
 
  <input type="submit" name="commit" value="Submit Post" />
</form>
```

#### The `#collection_check_boxes` helper

[This motherfucker](http://edgeapi.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html#method-i-collection_check_boxes) is pretty nifty. The syntax is a little complicated... but check it out:

```ruby
collection_check_boxes(method, collection, value_method, text_method, options = {}, html_options = {}, &block)
```

So you can use it like this:

```html
<%= form_for @post do |f| %>
  <%= f.collection_check_boxes :author_ids, Author.all, :id, :name_with_initial %>
  <%= f.submit %>
<% end %>
```

As far as I understand, it goes like this:

```ruby
collection_check_boxes(

    method,     # The method you call on @post to name
                # them, and which method it will call to
                # write the values to the @post object
                # (also the name of how they are stored in
                # params); they will look like this:

# <input type="checkbox" value="1" name="post[author_ids][]" id="post_author_ids_1">
# <label for="post_author_ids_1">J.K. Rowling</label>
# <input type="checkbox" value="2" name="post[author_ids][]" id="post_author_ids_2">
# <label for="post_author_ids_2">S. King</label>
# ... you get the idea

    collection,     # The collection that it populates the 
                    # checkboxes with
    value_method,   # The method you call on each object
                    # to set the value attr of the checkbox
    text_method,    # The method you call on each object
                    # to set the label for the field
    options = {},
    html_options = {},
    &block
)
```

See the `rails-blog-associations-validations-v-000` lab for an example of one in action (in the `app/views/posts/_form.html.erb` partial).

#### Collections of inputs

See the collection input helpers [here (and below)](http://edgeapi.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html#method-i-collection_check_boxes).

#### Other helpers

See [here](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html) for other form tag helper methods. There are quite a few!

### Making EVEN BETTER forms!

#### The `#form_for` method

You can use `#form_for` in order to create tailored forms more easily and more DRY. It accepts an instance of a model as an argument, and makes some assumptions from there: you don't have to make a bunch of calls to your `@post` instance variable or whatever, it automatically knows the standard route for the data, it even has an option for dynamically changing the submit button text, which can be useful for reasons. It yields an object of class `FormBuilder`, to which you can tack fields and stuff.

Basically, when you know you're gonna use a form directly on a model, use `#form_for`. When you just need a general form (search, contact, etc.), use `#form_tag`.

For example, we might have this:

```html
<%= form_tag post_path(@post), method: "put" do %>

  <label>Post title:</label><br>
  <%= text_field_tag :title, @post.title %><br>
 
  <label>Post description</label><br>
  <%= text_area_tag :description, @post.description %><br>
 
  <%= submit_tag "Submit Post" %>
  
<% end %>
```

...and we can use this instead:

```html
<%= form_for @post do |f| %>

  <label>Post title:</label><br>
  <%= f.text_field :title %><br>
 
  <label>Post description</label><br>
  <%= f.text_area :description %>
  
  <%= f.submit %>
  
<% end %>
```

Isn't that nice? And it automatically knows to use the `PATCH` method, because it's being passed a pre-existing record, as well as fill out the inputs with existing data. You'll have to change your route from `put` to `patch` since it uses `patch` by default, but that's solid.

Final example:

Use this...

```html
<%= form_for(@cat) do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %>
  <%= f.label :color %>
  <%= f.text_field :color %>
  <%= f.submit %>
<% end %>
```

...to generate this:

```html
<form accept-charset="UTF-8" action="/cats" method="post">
  <label for="cat_name">Name</label>
  <input id="cat_name" name="cat[name]" type="text" />
  <label for="cat_color">Color</label>
  <input id="cat_color" name="cat[color]" type="text" />
  <input name="commit" type="submit" value="Create" />
</form>
```

##### Errors and stuff

If you were to do this:

```html
<%= form_for @post do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit %>
<% end %>
```

Then the `FormBuilder#text_field` call will generate this:

```html
<input type="text" name="post[title]" id="post_title" value="Existing Post Title"/>
```

...which is great, but what's even cooler is that if there are errors for that attribute, it will make a `div.field_with_errors` around the offending field!

```html
<div class="field_with_errors">
  <input type="text" name="post[title]" id="post_title" value="Existing Post Title"/>
</div>
```

How cool is that!? Just remember that `div` is `display: block;` whereas `input` is `display: inline-block;`, so it could screw up styling.

##### Using `form_for` with nested resources

If you're posting to a nested resource (like `/authors/3/posts`) and you try to do:

```html
<%= form_for @post do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit %>
<% end %>
```

That's gonna give an error! Specifically, it will give a NoMethodError, saying it can't find `posts_path`. That's because the URL you want to post to is not `posts_path`, it's `author_posts_path(some_author)`. So instead, refactor it to something like this:

```html
<%= form_for @post, url: author_posts_path(@author) do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit %>
<% end %>
```

Yayyyyyy. You can actually shorten this syntax by passing `form_for` an array, like `[parent, child]`:

```html
<%= form_for [@author, @post] do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit %>
<% end %>
```

Rails will know, based on whether `@post` is new or saved in the database, if you're trying to `POST` to `author_posts_path` (`/authors/3/posts`) or `PATCH` to `author_post_path` (`/authors/3/posts/1`). Nice!

### Making `has_many` and `belongs_to` "association writers" for mass assignment

See [this lesson](https://github.com/learn-co-students/forms-and-basic-associations-rails-v-000) for a good overview of this concept.

#### `belongs_to` association writer

Basically, say you have two models:

```ruby
# app/models/post.rb
class Post < ActiveRecord::Base
  belongs_to :category
end
 
# app/models/category.rb
class Category < ActiveRecord::Base
  has_many :posts
end
```

Now, we build a form to create a post and choose a category for the post:

```html
<%= form_for @post do |f| %>
  <%= f.label :category_id, :category %>
  <%= f.text_field :category_id %>
  <%= f.text_field :content %>
<% end %>
```

That gives a form that allows you to automatically set `@post.category_id` and `@post.content` with the params via mass assignment. It will associate a category to a post, which is great. But wouldn't it be nicer, as a user of this form, to be able to type in the category's _name_, rather than its `id`?

Unfortunately, there's currently no way to set `@post.category` by name. So, while you can do this for the previous example:

```ruby
class PostsController < ApplicationController
  def create
    Post.create(post_params)
  end
 
  private
 
  def post_params
    params.require(:post).permit(:category_id, :content)
  end
end
```

...and have it work fine, you still have to use the `category_id` attribute which is clunky and requires a user to enter a number instead of the category's name.

Okay, so what if we make that field into `category_name` instead of `category_id`, and just handle it in the controller? Check it out:

```ruby
class PostsController < ApplicationController
  def create
    category = Category.find_or_create_by(name: params[:category_name])
    Post.create(content: params[:content], category: category)
  end
end
```

Now, that's great and works just fine, but we have to do this _everywhere_ we want to set a post's category. That's pretty terrible. We want mass assignment with params!

Turns out, if you define a setter method on a model, it will be called when you `create`/`update`/etc. an object with it as a key! Lemme show you. We define a `category_name` setter method on `Post`:

```ruby
# app/models/post.rb
class Post < ActiveRecord::Base
  belongs_to :category

  def category_name=(name)
    self.category = Category.find_or_create_by(name: name)
  end
end
```

Cool. So by doing `some_post.category_name = "Fitness"`, `some_post.category` will be a/the category named "Fitness". That's cool, but nothing new. However, if you `#create` a `Post` object like this:

```ruby
some_post = Post.create(
  category_name: "Fitness",
  content: "blah blah blah I love going to the gym."
)
```

Then `some_post.category` will be a/the category named "Fitness"! That's awesome. I didn't realize that those keys refer to the setter methods for those attributes—I thought ActiveRecord went straight to the database. But instead it looks like it _basically_ just does a `some_post.content = "blah blah blah"` and then a `some_post.save`. Cool.

So, finally, we change the view to reflect our new knowledge:

```html
<%= form_for @post do |f| %>
  <%= f.label :category_name %>
  <%= f.text_field :category_name %>
  <%= f.text_field :content %>
<% end %>
```

And our controller is now nice and simple:

```ruby
class PostsController < ApplicationController
  def create
    Post.create(post_params)
  end
 
  private
 
  def post_params
    params.require(:post).permit(:category_name, :content)
  end
end
```

Voilà!

#### `has_many` association writer

Okay, so what about selecting posts to add to a category? We run into the same problem, but this time there are multiple posts that can be associated to one category. Say we have a form that looks something like this:

```html
<%= form_for @category do |f| %>
  <input name="category[post_ids][]">
  <input name="category[post_ids][]">
  <input name="category[post_ids][]">
<% end %>
```

...where we get back an array of values in `params[:category][:post_ids]`, and we want to set them all in the category.

Well, we just write another setter method:

```ruby
# app/models/category.rb
class Category < ActiveRecord::Base
  has_many :posts
  
  def post_ids=(ids)
    ids.each do |id|
      post = Post.find(id)
      self.posts << post
    end
  end
end
```

And now our controller can be nice and simple:

```ruby
# app/controllers/categories_controller.rb
class CategoriesController < ApplicationController
  def create
    Category.create(category_params)
  end
 
  private
 
  def category_params
    params.require(:category).permit(:name, :post_ids => [])
  end
end
```

### Making nested forms

So, we have this neat way to assign stuff directly from forms with association writers (see last section). Cool. But what if you have, like, a `Person`, who has many `Address`es. Each `Address` has its own set of multiple attributes, and you can't just return it as an array and iterate over it with a `Person#address_ids=` method. What do you do?

Enter "nested attributes" on your `params` hash (with the `accepts_nested_attributes_for` macro), and the `fields_for` form helper method. I would love to explain this in detail, but it would take a bit of time. [This lesson](https://github.com/learn-co-students/basic-nested-forms-v-000) is a good one, and it'll go more in-depth than here.

Basically, you design your params hash using `"0"` and `"1"` as keys to a nested `addresses_attributes` hash, so one address is accessible like this: `params[:person][:addresses_attributes]["0"]`:

```ruby
{
  :person => {
    :name => "Avi",
    :email => "avi@flombaum.com",
    :addresses_attributes => {
      "0" => {
        :street_address_1 => "33 West 26th St",
        :street_address_2 => "Apt 2B",
        :city => "New York",
        :state => "NY",
        :zipcode => "10010",
        :address_type => "Work"
      },
      "1" => {
        :street_address_1 => "11 Broadway",
        :street_address_2 => "2nd Floor",
        :city => "New York",
        :state => "NY",
        :zipcode => "10004",
        :address_type => "Home"
      }
    }
  }
}
```

Rails will set this up for us if we use the `fields_for` FormHelper method, but we'll get to that in a sec. First, we use the `accepts_nested_attributes_for` macro in the `Person` model:

```ruby
class Person < ActiveRecord::Base
  has_many :addresses
  accepts_nested_attributes_for :addresses
end
```

That macro basically sets up a method `#addresses_attributes=` which will build new `Address` objects on the `my_person.addresses` collection, using the nested structure we saw earlier.

Now, if you were to do a `rails c` you would see that the following is true:

```bash
2.2.3 :018 > new_person = Person.new
 => #<Person id: nil, name: nil, created_at: nil, updated_at: nil>

2.2.3 :019 > new_person.addresses_attributes={"0"=>{"street_address_1"=>"33 West 26", "street_address_2"=>"Floor 2", "city"=>"NYC", "state"=>"NY", "zipcode"=>"10004", "address_type"=>"work1"}, "1"=>{"street_address_1"=>"11 Broadway", "street_address_2"=>"Suite 260", "city"=>"NYC", "state"=>"NY", "zipcode"=>"10004", "address_type"=>"work2"}}
 => {"0"=>{"street_address_1"=>"33 West 26", "street_address_2"=>"Floor 2", "city"=>"NYC", "state"=>"NY", "zipcode"=>"10004", "address_type"=>"work1"}, "1"=>{"street_address_1"=>"11 Broadway", "street_address_2"=>"Suite 260", "city"=>"NYC", "state"=>"NY", "zipcode"=>"10004", "address_type"=>"work2"}}

2.2.3 :020 > new_person.save
   (0.2ms)  begin transaction
  SQL (0.8ms)  INSERT INTO "people" ("created_at", "updated_at") VALUES (?, ?)  [["created_at", "2016-01-14 11:57:00.393038"], ["updated_at", "2016-01-14 11:57:00.393038"]]
  SQL (0.3ms)  INSERT INTO "addresses" ("street_address_1", "street_address_2", "city", "state", "zipcode", "address_type", "person_id", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)  [["street_address_1", "33 West 26"], ["street_address_2", "Floor 2"], ["city", "NYC"], ["state", "NY"], ["zipcode", "10004"], ["address_type", "work1"], ["person_id", 3], ["created_at", "2016-01-14 11:57:00.403152"], ["updated_at", "2016-01-14 11:57:00.403152"]]
  SQL (0.1ms)  INSERT INTO "addresses" ("street_address_1", "street_address_2", "city", "state", "zipcode", "address_type", "person_id", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)  [["street_address_1", "11 Broadway"], ["street_address_2", "Suite 260"], ["city", "NYC"], ["state", "NY"], ["zipcode", "10004"], ["address_type", "work2"], ["person_id", 3], ["created_at", "2016-01-14 11:57:00.405973"], ["updated_at", "2016-01-14 11:57:00.405973"]]
   (0.6ms)  commit transaction
 => true
```

Awesome! That macro set up a `Person#addresses_attributes=` method that works.

So, now, we just gotta get the form to make a hash like that:

```html
<!-- app/views/people/new.html.erb -->

<%= form_for @person do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %><br>

  <%= f.fields_for :addresses do |addr| %>
    <%= addr.label :street_address_1 %>
    <%= addr.text_field :street_address_1 %><br>

    <%= addr.label :street_address_2 %>
    <%= addr.text_field :street_address_2 %><br>

    <%= addr.label :city %>
    <%= addr.text_field :city %><br>

    <%= addr.label :state %>
    <%= addr.text_field :state %><br>

    <%= addr.label :zipcode %>
    <%= addr.text_field :zipcode %><br>

    <%= addr.label :address_type %>
    <%= addr.text_field :address_type %><br>
  <% end %>

  <%= f.submit %>
<% end %>
```

How cool is that? It makes fields for the `addresses` collection on `@person`, and it will automatically return the address attribute hashes the way we wanted them to be returned earlier.

Unfortunately, there's one last thing we have to do. If you were to load that page with the "new" form, you wouldn't see anything for the addresses. You _would_, if it were the "edit" form, and the `Person` already _had_ addresses, but there are currently no addresses for the form to see, so it won't build forms for them. Solve this by stubbing out new addresses in the controller so the form has something to build (the same way you need `@person = Person.new` to build the "new" form to begin with):

```ruby
class PeopleController < ApplicationController
  def new
    @person = Person.new
    @person.addresses.build(address_type: 'work')
    @person.addresses.build(address_type: 'home')
  end

  def create
    person = Person.create(person_params)
    redirect_to people_path
  end

  def index
    @people = Person.all
  end

  private

  def person_params
    params.require(:person).permit(:name)
  end
end
```

Nice. Two new address forms.

One last thing: if you notice, hitting "Submit" doesn't do anything. Why? You haven't `permit`ted the addresses' attributes! Do this:

```ruby
def person_params
  params.require(:person).permit(
    :name,
    addresses_attributes: [
      :street_address_1,
      :street_address_2,
      :city,
      :state,
      :zipcode,
      :address_type
    ]
  )
end
```

Remember how if you want to pass an array you have to do something along the lines of:

```ruby
params.require(:user).permit(:name, :some_array => [])
```

Well, for the `addresses_attributes`, you have to fill that array with the names of keys. I guess if you just have an array, you are telling `permit` that the array "has no keys", and if you want to pass it something with keys, you have to explicitly permit them in the given array.

Unfortunately, this doesn't avoid duplicates. Like, two people might both live at 138 Kidder Ave, so maybe it makes sense to have two rows in your addresses database for 138 Kidder Ave, so you're fine. But, if we wanted to use it on an artist for a song, you'd want to use a `find_or_create_by` in an association writer method instead, so that it  doesn't make duplicate artists.

#### One address, without pre-building in the controller

If you wanted `fields_for` one address, you can leave out the lines in the controller that build stub addresses and do this in your view instead:

```html
<%= form_for @person do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %><br>

  <%= f.fields_for :addresses, @person.addresses.build do |addr| %>
    <%= addr.label :street_address_1 %>
    <%= addr.text_field :street_address_1 %><br>

    <%= addr.label :street_address_2 %>
    <%= addr.text_field :street_address_2 %><br>

    <%= addr.label :city %>
    <%= addr.text_field :city %><br>

    <%= addr.label :state %>
    <%= addr.text_field :state %><br>

    <%= addr.label :zipcode %>
    <%= addr.text_field :zipcode %><br>

    <%= addr.label :address_type %>
    <%= addr.text_field :address_type %><br>
  <% end %>

  <%= f.submit %>
<% end %>
```

(emphasis on line 5)

See, you can have `fields_for` iterate over each existing address in `@post.addresses`, like in the last section, or you can specify an object to be added to that collection (`@person.addresses.build`), like here.

Or, as another example, say you want to make a post, select from existing categories, and/or create a new category:

```html
<%= form_for post do |f| %>
  <%= f.label "Title" %>
  <%= f.text_field :title %>
  <%= f.label "Content" %>
  <%= f.text_area :content %>
  <%= f.collection_check_boxes :category_ids, Category.all, :id, :name %>
  <%= f.fields_for :categories, post.categories.build do |categories_fields| %>
    <%= categories_fields.text_field :name %>
  <% end %>
  <%= f.submit %>
<% end %>
```

That `fields_for` line generates the following html:

```html
<input type="text" name="post[categories_attributes][0][name]" id="post_categories_attributes_0_name">
```

If we don't want category dupes from our newly-created categories, override the `#categories_attributes` method:

```ruby
class Post < ActiveRecord::Base
  has_many :post_categories
  has_many :categories, through: :post_categories
  accepts_nested_attributes_for :categories
 
  def categories_attributes=(category_attributes)
    category_attributes.values.each do |category_attribute|
      category = Category.find_or_create_by(category_attribute)
      self.categories << category
    end
  end
end
```

Note that we use the `category_attributes.values` as the enumerable, since it's actually a hash (rather than an array, which it seems like it should be). The `category_attribute` passed in is a hash which looks like this: `{"name"=>"Some Category"}`.

#### Rejecting the nested attributes if blank (or some other condition)

What if you want the new address or category field to be able to be left blank? If it's optional, it should be able to be left blank and not create some empty object on the receiving end.

Use `reject_if` to accomplish this task:

```ruby
class Person < ActiveRecord::Base
  has_many :addresses
  accepts_nested_attributes_for :addresses, reject_if: :all_blank
end
```

The `:all_blank` symbol checks to see if all the attributes on the nested attributes hash are blank, and `accepts_nested_attributes_for` will be rejected if so.

You can also specify a proc for more granular control:

```ruby
class Post < ActiveRecord::Base
  has_many :categories
  accepts_nested_attributes_for :categories, reject_if: proc { |attributes| attributes[:name].blank? }
end
```

You can also pass it a symbol referring to the name of a method:

```ruby
class Post < ActiveRecord::Base
  has_many :categories
  accepts_nested_attributes_for :categories, reject_if: :category_name_blank?
  
  def :category_name_blank?(attributes)
    attributes[:name].blank?
  end
end
```

Which works with existing methods, too, remember:

```ruby
class Post < ActiveRecord::Base
  has_many :categories
  accepts_nested_attributes_for :categories, reject_if: :new_record?
end
```

Awesome.

### Making form-wrapped buttons with `button_to`

Say you have a To-Do app, and you want to have a button that deletes an item from a list. You'd normally have to wrap that button in a form tag:

```html
<!-- 
picture this nested in an iterator, passing in items from
some particular list, building a <ul> with <li>s, with this
button on each item so you can delete it.
-->

<%= form_tag list_item_path(item.list, item), method: :delete do %>
  <button type="submit" class="destroy">x</button>
<% end %>
```

Instead, you can use a shorter syntax:

```html
<%= button_to "x", list_item_path(item.list, item), method: "delete", class: "destroy" %>
```

This is a nice way to delete items through RESTful conventions without resorting to javascript!

## The asset pipeline

### The basics

Apparently [this keynote](https://www.youtube.com/watch?v=cGdCI2HhfAU) where DHH (creator of Rails) introduces the Asset Pipeline is a good watch. I'll watch it later.

### Asset paths

Paths to asset folders are configured like this:

```ruby
Rails.application.config.assets.paths => [
  "/Users/avi/asset-test/app/assets/images",
  "/Users/avi/asset-test/app/assets/javascripts",
  "/Users/avi/asset-test/app/assets/stylesheets",
  "/Users/avi/asset-test/vendor/assets/javascripts",
  "/Users/avi/asset-test/vendor/assets/stylesheets",
  "/Users/avi/.rvm/gems/ruby-2.2.3/gems/turbolinks-2.5.3/lib/assets/javascripts",
  "/Users/avi/.rvm/gems/ruby-2.2.3/gems/jquery-rails-4.1.0/vendor/assets/javascripts",
  "/Users/avi/.rvm/gems/ruby-2.2.3/gems/coffee-rails-4.1.1/lib/assets/javascripts"
]
```

If you want to add folders to the asset pipeline, add them like this in `config/initializers/assets.rb`:

```ruby
Rails.application.config.assets.paths << "New Path"
```

This way, we can put assets anywhere and access them via a single `/assets` URL, kinda like the `$PATH` system variable.

The asset path is used for JS, CSS, and images.

#### Getting the path with the `#asset_path` helper

Of course this exists:

```ruby
asset_path('logo.png')
#=> "/assets/logo.png" (or wherever it is)

# if it's in a subdirectory not in the assets path...
asset_path('banner/logo.png')
#=> "/assets/banner/logo.png"

# if it can't find the file in the asset path:
asset_path('logo.png')
#=> "/logo.png"
```

Very useful for CSS:

```
.logo {
  background-image: url(<%= asset_path('logo') %>);
}
```

You don't have to use the helper with `#image_tag` (it uses it automatically):

```html
<%= image_tag "logo.png" %>
```

...results in:

```html
<img src="/assets/logo.png" />
```

...and if it can't be found? Again, it just puts it at `/logo.png`.

### Manifest files

Just having stuff in your path doesn't mean it will be _used_ in your application. If you want that (say, a js calendar file you added a path to), you gotta add the asset to the manifest file:

```js
// app/assets/javascripts/application.js

//= require jquery
//= require calendar
```

From [this Learn lesson](https://github.com/learn-co-students/what-is-the-asset-pipeline-v-000):

> When you include the manifest file in your layout with the javascript_include_tag, the asset pipeline will look for all of the files listed in the Asset Path. Notice how we require calendar. This file lives in app/assets/javascripts/calendar.js, yet we only specified the name and not the full path. The Asset Pipeline will search all the configured paths for a file with the name we provided.

Rails concatenates all these files together when they need to be served.

#### Directives

In the javascript manifest file, you can see `//=` statements. Those are called "directives". They're not normal JS comments; they tell sprockets to do something.

Notes on the `require` directive from [this Learn lesson]():

> You may notice another directive in your manifest file. The `require_tree` directive tells sprockets to load all files in the given folder. By default, the manifest file has `//= require_tree` . which will include all JS files in the same folder that `application.js` is located. This makes adding new JS files into our application really easy but can cause problems. As your application grows, you may not want all of the JS loaded everywhere. For example, say you have two pages that have a similiar button. You want those two buttons to behave differently even though they look the same. If all JS is loaded all the time, then the browser will not know which JS should be applied to each button. In the end, it's generally better to control which JS is being loaded for each page. Additionally, the files will be loaded in alphabetical order. Often, external libraries will have a dependency on another JS file being loaded before it, and all your JavaScript will error out if this load order is not honored. Given that things load in alphabetical order, it's unlikely things will magically be loaded in the right order. Check the console in your browser to see if you are getting these types of errors, and, if so, manually `require` each file in the order you need it to be loaded and get rid of `require_tree`.
> 
> One last thing to remember: when you `require` something in your manifest file, the path you provide must be the asset path. If you have the file `comments.js` in the folder `/assets/javascripts/blog`, you would need to use `//= require blog/comments` to include it in our manifest file.

CSS directives are similar, but they look like this:

```css
/*
*= require application
*= require main
*/
```

### Preprocessors

Rails lets you use SCSS and CoffeeScript and preprocesses assets before they are served. Just name `main.css.scss`, for example, and Rails will turn that into `main.css` for ya.

#### Precompiling for production

From [this Learn lesson](https://github.com/learn-co-students/asset-preprocessors-in-rails-v-000):

>In development mode, the asset pipeline will run the preprocessor on any file that needs to be processed every time it's requested. This is SLOW! In production, you might have had to run `rake assets:precompile`. This runs the preprocessors once, outputs all the now-static JS and CSS, and then allows the webserver to serve them without ever touching Rails. This is much quicker!


### Fingerprinting

For cache invalidation. You know what this is. I won't waste my time writing it down.

### Finally, including in the view

Do this:

```html
<%= javascript_include_tag "application" %>
```

...and sprockets will compile the manifest. In development mode, it will insert/load each file separately. In production mode, it will compile them all into one big file.

For CSS, basically the same thing:

```html
<%= stylesheet_link_tag "application" %>
```

It's a good idea to have separate JS/CSS manifest files for each controller/action/function/whatever. That way you aren't sending allllll of the app's JS logic for every single little thing. If you name them for your controllers (and add them to the asset load path in `config/initializers/assets.rb`) you can do this:

```html
<%= javascript_include_tag params[:controller] %>
```

Yeah. You can do that. `params` has the controller name and the action name right there for you to grab. How cool is that? How have you never noticed that? You IDIOT! YOU STUPID PIECE OF SHIT! WHO DO YOU THINK YOU ARE!? Ahem.

But, from [this Learn lesson](https://github.com/learn-co-students/page-specific-javascript-rails-v-000):

>The downside of this is we'd no longer be getting the benefits of asset concatenation or caching. The browser will have to make a separate request for this file in addition to the request for the main concatenated application.js file. The benefit of this strategy could be that we're less likely to invalidate the cache for our entire JS file if it is in pieces.

There are other methods of SoC'ing your JS listed on that lesson page. Probably helpful to know them.

## Miscellaneous useful gems

### Using Bootstrap (gem method)

Put this in your Gemfile:

```ruby
gem 'bootstrap-sass'
```

Run `bundle`, then put this in your CSS (SCSS) manifest file:

```css
@import "bootstrap-sprockets";
@import "bootstrap";
```

Then put this in your JS manifest file:

```js
//= require bootstrap-sprockets
```

Noice. Now, any time you want to update Bootstrap, all ya gotta do is run `bundle update`.

### File attachment/uploading with Paperclip

See [this lesson](https://github.com/learn-co-students/rails-paperclip-readme-v-000) for a good overview of using the Paperclip gem to handle attaching files and uploads (e.g., adding an avatar image to a user record in the DB).

### Pagination with Kaminari

See [this lesson](https://github.com/learn-co-students/rails-kaminari-readme-v-000). Really great and simple pagination. Like crazy simple. So good.

### Stellar + easy admin dashboard with Active Admin

Holy balls. See [this lesson](https://github.com/learn-co-students/rails-active-admin-readme-v-000). So much incredible functionality for so little effort. Amazing.

### Long-running tasks

#### Long-running CSV processing

See [this lesson](https://github.com/kjleitz/rails-long-running-tasks-readme-v-000). Pretty basic. But a decent overview I think.

#### Moving to a worker (using Sidekiq)

Picture doing that on a CSV file containing a bunch of song info, for example, with thousands of records. That takes a while. Move the logic to a worker (using Sidekiq) to run in the background:

Add:

```
gem 'sidekiq'
```

...to your Gemfile. Install `redis` if you haven't already:

```
$ brew install redis
```

Then launch (and restart at login):

```
$ brew services start redis
```

Create a worker by adding an `app/workers` directory and making a file like this:

```ruby
# app/workers/songs_worker.rb

class SongsWorker
  include Sidekiq::Worker
 
  def perform(songs_file)
 
  end
end
```

You give it an instance method of `#perform` which takes the data you need to operate on as an argument. Slap that logic in there:

```ruby
# app/workers/songs_worker.rb

class SongsWorker
  require 'csv'
  include Sidekiq::Worker
 
  def perform(songs_file)
    CSV.foreach(songs_file, headers: true) do |song|
      Song.create(title: song[0], artist_name: song[1])
    end
  end
end
```

Back in our controller, in the song upload action, we can use the worker to perform the logic instead of doing it in the action itself (meaning it will no longer be blocking and synchronous):

```ruby
# app/controllers/songs_controller.rb

class SongsController < ApplicationController
  
  # ...
 
  def upload
    SongsWorker.perform_async(params[:songs].path)
    redirect_to customers_path
  end
end
```

Now, all ya gotta do is make sure redis is started (should have started if you did `brew services start redis`, and it _should_ start at login after that, but do `redis-server` I guess if not) and then start sidekiq by doing:

```
$ bundle exec sidekiq
```

...then start your Rails server, and you should be all set!

### Making requests/API calls with Faraday

Faraday is actually a pretty nifty little gem for wrapping some lower-level HTTP request libs. You can make and build requests similar to how you would using, say, Postman. This is how you might do some Foursquare searches with Faraday.

Add to your Gemfile:

```ruby
gem 'faraday'
```

Create an HTML form for the search (for coffee shops):

```html
<!-- app/views/searches/search.html.erb -->

<h1>Search for Coffee Shops Near Me</h1>
<%= form_tag '/search' do %>
  <%= label_tag :zipcode %>
  <%= text_field_tag :zipcode %>
  <%= submit_tag "Search!" %>
<% end %>
```

In your controller:

```ruby
# app/controllers/searches_controller.rb

class SearchesController < ApplicationController
  def search
  end

  def foursquare
    resp = Faraday.get 'https://api.foursquare.com/v2/venues/search' do |req|
      req.params['client_id'] = client_id
      req.params['client_secret'] = client_secret
      req.params['v'] = '20160201'
      req.params['near'] = params[:zipcode]
      req.params['query'] = 'coffee shop'
    end
    
    body_hash = JSON.parse(resp.body)
    @venues = body_hash['response']['venues']
    render 'search'
  end
end
```

Cool, right? `resp` now has a `#body` and `#status` (among other things) which are, respectively, the returned JSON data and the HTTP status code. Dig into the JSON to get what you want out of it (the venue list).

We can change our view to include this data:

```html
<!-- app/views/searches/search.html.erb -->

<h1>Search for Coffee Shops Near Me</h1>
<%= form_tag '/search' do %>
  <%= label_tag :zipcode %>
  <%= text_field_tag :zipcode %>
  <%= submit_tag "Search!" %>
<% end %>

<div>
  <% if @venues %>
    <ul>
    <% @venues.each do |venue| %>
      <li>
        <%= venue["name"] %><br>
        <%= venue["location"]["address"] %><br>
        Checkins: <%= venue["stats"]["checkinsCount"] %>
      </li>
    <% end %>
    </ul>
  <% end %>
</div>
```

Great! But what if we get an error back? If the request is malformed, it'll send a status code of `400` (bad request) back. We can check to see if the status code is `200` directly, or use a nice little abstracted method:

```ruby
# app/controllers/searches_controller.rb

class SearchesController < ApplicationController
  def search
  end

  def foursquare
    resp = Faraday.get 'https://api.foursquare.com/v2/venues/search' do |req|
      req.params['client_id'] = client_id
      req.params['client_secret'] = client_secret
      req.params['v'] = '20160201'
      req.params['near'] = params[:zipcode]
      req.params['query'] = 'coffee shop'
    end
    
    body_hash = JSON.parse(resp.body)
    if @resp.success?
      @venues = body_hash["response"]["venues"]
    else
      @error = body_hash["meta"]["errorDetail"]
    end
    render 'search'
  end
end
```

Noice. Then just put some div to handle the error in the view, or whatever.

What about a timeout?

```ruby
# app/controllers/searches_controller.rb

class SearchesController < ApplicationController
  def search
  end

  def foursquare
    resp = Faraday.get 'https://api.foursquare.com/v2/venues/search' do |req|
      # ...
      req.options.timeout = 500
    end
    
    # ...
    
    rescue Faraday::TimeoutError
      @error = "There was a timeout. Please try again."
    end
    
    render 'search'
  end
end
```

Neat. Set the timeout time you want, handle the error if it's over the limit.

#### OAuth with Faraday

There's some decent stuff in [this Learn lesson](https://github.com/kjleitz/web-auth-readme-v-000).

#### Moving this crap out into Service Objects

That shit doesn't belong in the controller. Put it in a service or whatever, like [this](https://github.com/learn-co-students/web-service-objects-readme-v-000). Allllllll that logic should go into a service object/class.

## Ajax & Rails

See the [Ajax notes](ajax.md), [XMLHttpRequest notes](xmlhttprequest.md), and the [JavaScript notes](javascript.md) for more deets. Here are a few quick tips:

## Building an API

### Returning HTML

If you grab a post's comments by hitting `/posts/3/comments`, you'll get the layout's HTML back, too. You can disable layouts from being rendered with an explicit render:

```ruby
# app/controllers/comments_controller.rb

# ...
  render 'comments/index', layout: false
# ...
```

You can actually just do this to render implicitly:

```ruby
# app/controllers/comments_controller.rb

# ...
  render layout: false
# ...
```

Then you can just drop that HTML in like bam.

### Returning JSON

You can return JSON instead of HTML at the end of an action like this:

```ruby
render json: @comments
```

Literally the easiest thing. Turns that collection right into JSON. Wow.

Also, if there's a request (like, imagine doing an Ajax request) and the content type it specifies is JSON, Rails will actually respond with the same-named file with a JSON extension in your views folder. Same as if the content type it requests is `script`, it will look for a JS file to return:

```
app
 +-- controllers
 +-- models
 +-- views
      +-- comments
           +-- index.html.erb
           +-- index.json.erb
           +-- index.js.erb
      +-- posts
      +-- users
 +-- ...
```

You can make this explicit by doing this:

```ruby
respond_to do |format|
  format.html { render 'index.html', layout: false }
  format.js { render 'index.js', layout: false }
end
```

Cool.

If you want a link to load a JS file by an AJAX request at its `href` instead of the HTML resource at the same location, you can take a shortcut:

```html
<%= link_to "Comments", comments_path %>
<!-- redirects and renders comments/index.html  -->

<%= link_to "Comments", comments_path, remote: true %>
<!-- fetches comments/index.js w/ Ajax and prevents reload -->
```

That's called the "remote true" or "get script" pattern. Only in Rails. Kinda antithetical to using a framework like React or Angular or Ember. Basically you have like zero actual front-end JS, it just fetches JS from the server when you need the logic for whatever thing needs to happen.

### Serializing objects

Basically, making data into a string. Make an `app/serializers` directory with `<model>_serializer.rb` files and keep that shit out of the controllers. Use [this lesson](https://github.com/learn-co-students/diy-json-serializer-ruby-v-000) as reference.

On the other hand, you can use `<model_object>.to_json` to do the same thing, which is awesome. You can even tell it to include properties which are relations of it (e.g., `post.to_json(include: :author)`). You can exclude stuff, too... check it out:

```ruby
post.to_json(only: [:title, :description, :id], include: [ author: { only: [:name]}])

# or, prettier:

post.to_json(
  only: [
    :title,
    :description,
    :id
  ],
  include: [
    author: {only: [:name]}
  ]
)
```

...which would spit out:

```json
{
  id: 1,
  title: "A Blog Post By Stephen King",
  description: "This is a blog post by Stephen King. It will probably be a movie soon.",
  author: {
    name: "Stephen King"
  }
}
```

### Serializing with ActiveModel::Serializer (AMS)

That `to_json` call can get kinda hairy. Using AMS is more like creating your own serializer, but it will serialize implicitly so it doesn't uglify your code! Noice. Also look up jbuilder if you get the chance (but it ties JSON templating to the views, whereas AMS ties it to the controllers... I like the latter approach, because it fits more with creating a back-end that's just a clean JSON API).

Using AMS allows you to specify what you want to be inplicitly sent if you use `render json: <model_object>`.

#### Creating/using a serializer

Add it to your Gemfile (if you're still on Rails 4):

```ruby
gem 'active_model_serializers'
```

Then, `bundle install`. Run the following command to generate a serializer for your `Post` model:

```
$ rails g serializer post
```

That'll puke up something like this:

```ruby
# app/serializers/post_serializer.rb

class PostSerializer < ActiveModel::Serializer
  attributes :id
end
```

Nice. Give it some more attributes from your actual `Post` model:

```ruby
class PostSerializer < ActiveModel::Serializer
  attributes :id, :title, :description
end
```

Now, back in your `posts_controller.rb`, you can get rid of that `to_json` call:

```ruby
  def show
    @post = Post.find(params[:id])
    respond_to do |format|
      format.html { render :show }
      format.json { render json: @post}
    end
  end
```

Et voilà! What whaaaaaat. Now, when you navigate to `/posts/1.json`, you'll get the following:

```json
{
  id: 1,
  title: "A Blog Post By Stephen King",
  description: "This is a blog post by Stephen King. It will probably be a movie soon."
}
```

Hell yeah.

#### Including relationships in the serialized data

What if that `Post` has an associated `Author` model? Don't you want that showing up in your JSON? Check it out:

```
$ rails g serializer author
```

Now you have an `Author` serializer. Go ahead and add the `name` attribute to it, like you did with the other attributes on `Post`. Then, specify the relationships:

```ruby
class PostSerializer < ActiveModel::Serializer
  attributes :id, :title, :description
  belongs_to :author
end
```

Now, if you navigate to `/posts/1.json` you'll see this:

```json
{
  id: 1,
  title: "A Blog Post By Stephen King",
  description: "This is a blog post by Stephen King. It will probably be a movie soon.",
  author: {
    id: 1,
    name: "Stephen King"
  }
}
```

YEAH! That's IT! How cool is that?

You can even do this:

```ruby
class AuthorSerializer < ActiveModel::Serializer
  attributes :id, :name
  has_many :posts
end
```

...take a guess at what'll happen.

Even better (last thing, I promise): say we don't care about the nested author's `id` when serving the `Post`'s JSON. We _could_ remove that attribute from the `AuthorSerializer`, but then what if we want to serve JSON of an author alone? We'd be taking it out of that, too. Instead, we can make a more specific serializer and explicitly use it to represent the `Post`'s author data:

```
$ rails g serializer post_author
```

...then we give the resulting serializer _just_ the author's name (all we want):

```ruby
class PostAuthorSerializer < ActiveModel::Serializer
  attributes :name
end
```

...and then, in our `PostSerializer`, we explicitly tell it to use the new serializer for the author:

```ruby
class PostSerializer < ActiveModel::Serializer
  attributes :id, :title, :description
  belongs_to :author, serializer: PostAuthorSerializer
end
```

Now, by navigating to `/posts/1.json`, you should get back:

```json
{
  id: 1,
  title: "A Blog Post By Stephen King",
  description: "This is a blog post by Stephen King. It will probably be a movie soon.",
  author: {
    name: "Stephen King"
  }
}
```

Awesome.

#### Serializing form data to POST with Ajax (jQuery)

Say you have a form that you want to submit with Ajax, rather than having it reload the page and all that jazz. jQuery gives you a really useful `serialize` method that will automatically serialize the form for you (which you can pass `this` in the event handler because it refers to the element being clicked on! Oh **boy!!!**):

```js
  $(function () {
    $('form').submit(function(event) {
      //prevent form from submitting the default way
      event.preventDefault();
      var values = $(this).serialize();
      var posting = $.post('/posts', values);
 
      posting.done(function(data) {
        // TODO: handle response
      });
    });
  });
```

Coooooool. That will put the data into JSON and POST it to `/posts` as params. Of course, you're gonna want to properly send stuff back afterward, because if you leave a redirect in the controller action you trigger, it will still redirect even though you tried to `e.preventDefault()`. If you send a JSON representation of the object back as a response, instead, you can do this with it:

```js
$(function () {
    $('form').submit(function(event) {
      //prevent form from submitting the default way
      event.preventDefault();
      var values = $(this).serialize();
      var posting = $.post('/posts', values);
 
      posting.done(function(data) {
        var post = data;
        $("#postTitle").text(post["title"]);
        $("#postBody").text(post["description"]);
      });
    });
  });
```

Now it creates the resource and displays the newly created data all without refreshing the page, and without "faking" showing the new data (as in, changing it to the "new" values without actually checking if it successfully went through on the server).

### Serving different formats (HTML vs. JSON, etc.)

Use a `respond_to` block:

```ruby
# posts_controller
# ...
  def show
    @post = Post.find(params[:id])
    respond_to do |format|
      format.html { render :show }
      format.json { render json: @post.to_json(only: [:title, :description, :id], include: [author: { only: [:name]}]) }
    end
  end
```

You can browse to `/posts/3` and get HTML back (default behavior), or to `/posts/3.html` for the same thing. Or, you can browse to `/posts/3.json` and it will return the JSON! You can also specify the content type you want in the headers of the request (`'Accept'` as `application/json`, I believe).

### Specifying status codes

You wanna be granular in your status codes for an API:

```ruby
  def create
    @post = Post.create(post_params)
    render json: @post, status: 201
  end
```

`201` goes further than a normal `200` (OK) status—it tells you that the resource was `created`!