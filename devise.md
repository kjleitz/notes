# Devise notes

---

[toc]

---

## User model abilities

_This is mostly taken from [here](https://github.com/learn-co-students/devise_readme-v-000)._

Devise is made up of modules, which are applied to your `User` model and give them certain new "abilities":

- **Database Authenticatable:** encrypts and stores a password in the database to validate the authenticity of a user while signing in. The authentication can be done both through `POST` requests or HTTP Basic Authentication.
- **Omniauthable:** adds OmniAuth support.
- **Confirmable:** sends emails with confirmation instructions and verifies whether an account is already confirmed during sign in.
- **Recoverable:** resets the user password and sends reset instructions.
- **Registerable:** handles signing up users through a registration process, also allowing them to edit and destroy their account.
- **Rememberable:** manages generating and clearing a token for remembering the user from a saved cookie.
- **Trackable:** tracks sign in count, timestamps and IP address.
- **Timeoutable:** expires sessions that have not been active in a specified period of time.
- **Validatable:** provides validations of email and password. It's optional and can be customized, so you're able to define your own validations.
- **Lockable:** locks an account after a specified number of failed sign-in attempts. Can unlock via email or after a specified time period.

## Setting up Devise

Add `gem 'devise'` to your Gemfile, then run the installer:

```
$ rails generate devise:install
```

This creates a big initializer file under `config/initializers/devise.rb`, which is actually super useful documentation for how it works. It will also print a big message detailing what you have to set up.

For one, you need to have a root route, so create a `WelcomeController` with a `#home` view and route.

Then, generate a Devise `User` model:

```
$ rails generate devise User
```

If you run `rake routes`, you'll see that Devise has added a bunch of routes for us. If you run `rails s` and navigate to `/users/sign_in` you can see it has set it up already. Link to it in a view like this:

```html
<!-- views/layouts/application.html.erb -->
 
  ...
 
  <%= link_to("Sign Up", new_user_registration_path) %>
 
  ...
```

You can also see that it has set up `/users/sign_out`, so you can implement that in a view like so:

```html
<!-- views/layouts/application.html.erb -->
 
  ...
 
  <%= link_to("Sign Out", destroy_user_session_path) %>
 
  ...
```

It also supplies flash messages when a user signs in or if something goes wrong, which you can access in a view as well:

```html
<!-- views/layouts/application.html.erb -->
 
  ...
 
  <%= content_tag(:div, flash[:error], :id => "flash_error") if flash[:error] %>
  <%= content_tag(:div, flash[:notice], :id => "flash_notice") if flash[:notice] %>
  <%= content_tag(:div, flash[:alert], :id => "flash_alert") if flash[:alert] %>
 
  ...
```

## Generating a Devise `User`

Run the following command:

```
$ rails generate devise User
```

...which will generate a nice `User` model with some helpful notes:

```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
end
```

## Helpful methods on the Devise `User`

### database_authenticatable

Adds `User#valid_password?(password)` - the password is stored securely in the database.

### registerable

Adds `User::new_with_session(params, session)` - lets you initialize a `User` from `session` data (like a token from Facebook) in addition to `params`.

### recoverable

Adds password resets:

```ruby
# reset the user password and save the record, 
# true if valid passwords are given, otherwise false
User.find(1).reset_password('password123', 'password123')
 
# only resets the user password, without saving the record
user = User.find(1)
user.reset_password('password123', 'password123')
 
# creates a new token and sends it with instructions 
# on how to reset the password (requires a mailer.)
User.find(1).send_reset_password_instructions
```

### rememberable

Lets you remember a user and associate them with a User object in the database without them having to log in. It works by storing a token in cookies:

```ruby
User.find(1).remember_me!  # regenerating the token
User.find(1).forget_me!    # clearing the token
```

### trackable

Track information about your user's sign-ins. It tracks the following columns:

- `sign_in_count` — Increased every time a user signs in (by form, OpenID, OAuth, etc.)
- `current_sign_in_at` — A timestamp updated when the user signs in
- `last_sign_in_at` — Holds the timestamp of the previous sign-in
- `current_sign_in_ip` — The remote IP updated when the user signs in
- `last_sign_in_ip` — Holds the remote IP of the previous sign-in

### validatable

From the docs:

"Validatable creates all needed validations for a user email and password. It's optional, given you may want to create the validations by yourself. Automatically validate if the email is present, unique and its format is valid. Also tests presence of password, confirmation and length."

### lockable

Handles blocking a user's access after a certain number of attempts. Lockable accepts two different strategies to unlock a user after they're blocked: email and time. The former will send an email to the user when the lock happens, containing a link to unlock their account. The second will unlock the user automatically after some configured time (e.g., two hours). It's also possible to set up Lockable to use both email and time strategies.

### omniauthable

Sets some routines for OmniAuth, but not much new stuff.

## Omniauth with Devise

Add this to your Gemfile:

```ruby
gem 'omniauth-facebook'
```

Add a provider and uid to your `User` model:

```
$ rails g migration AddOmniauthToUsers provider:index uid:index
$ rake db:migrate
```

Add an OmniAuth config line to `config/initializers/devise.rb`:

```ruby
config.omniauth :facebook, ENV['FACEBOOK_KEY'], ENV['FACEBOOK_SECRET']
```

And set your environment variables up as described in the appropriate section in the [Rails notes](rails.md) after creating a [Facebook app](https://developer.facebook.com/). Then, in the developer console for the application on Facebook, click the "Facebook Login" button in the sidebar and add `http://localhost:3000/users/auth/facebook/callback` to your Valid OAuth Redirect URLs. Then, add this line to your `User` model:

```ruby
devise :omniauthable, :omniauth_providers => [:facebook]
```

Now, you can link to the Facebook login like so:

```html
<%= link_to "Sign in with Facebook", user_facebook_omniauth_authorize_path %>
```

Now, if you try to log in, you'll be greeted with an error: "The action 'facebook' could not be found for Devise::OmniauthCallbacksController". So, add this to your `routes.rb` file:

```ruby
devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }
```

Now, you'll need to make a file at `app/controllers/users/omniauth_callbacks_controller.rb`:

```ruby
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def facebook
    # TODO
  end
end
```

(The action should have the same name as the provider)

So, we'll want to handle the OmniAuth callback from facebook:

```ruby
def facebook
  @user = User.from_omniauth(request.env["omniauth.auth"])
  sign_in_and_redirect @user      
end
```

That `User#from_omniauth` method will need to be written, too:

```ruby
class User < ActiveRecord::Base
  def self.from_omniauth(auth)
    where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
      user.email = auth.info.email
      user.password = Devise.friendly_token[0,20]
    end      
  end
end
```

(gives the user a safe dummy password)

You'll get a missing template error. That's because `sign_in_and_redirect` needs to know what to do. Put this in your `ApplicationController`:

```ruby
def after_sign_in_path_for(resource)
  request.env['omniauth.origin'] || root_path
end
```

Jesus Christ.