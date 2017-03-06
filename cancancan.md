# CanCanCan notes

CanCanCan is a gem which lets you _authorize_ (not authenticate) users to perform actions.

## Basic usage

You can use CanCanCan in your controllers like this:

```ruby
def show
  @article = Article.find(params[:id])
  authorize! :read, @article
end
```

Here we explicitly use `#authorize!` to determine if a user can read this article. If they can't, an exception is thrown.

This can get pretty un-DRY after a while. So, you can authorize all actions in a controller by using the `load_and_authorize_resource` macro to authorize all actions in a controller by using a `before` filter to load the resource into an instance variable and authorize it for every action:

```ruby
class ArticlesController < ApplicationController
  load_and_authorize_resource
 
  def show
    # @article is already loaded and authorized
  end
end
```

## Failed authorization

CanCanCan will throw a `CanCan::AccessDenied` exception if a user is not authorized to access a resource, which you can handle like this:

```ruby
class ApplicationController < ActionController::Base
  rescue_from CanCan::AccessDenied do |exception|
    redirect_to root_url, :alert => exception.message
  end
end
```

## In your views

This is pretty cool, I think:

```html
<% if can? :update, @article %>
  <%= link_to "Edit", edit_article_path(@article) %>
<% end %>
```

Pretty self-explanatory. Also this:

```html
<% if cannot? :update, @article %>
  Editing disabled.
<% end %>
```

Neat. Love the syntax.

## Abilities

### The basics

You can generate an `Ability` class with:

```
$ rails g cancan:ability
```

Fill out the ability class like so:

```ruby
class Ability
  include CanCan::Ability

  def initialize(user)
    if user.admin?
      # only admins can change things
      can :update, Article
    end
    # but anyone can read them
    can :read, Article
  end
end
```

Pretty self-explanatory, again. I like this gem. But for posterity's sake:

The class takes an instance of `User` and defines what it can and cannot do, using the methods `#can` and `#cannot`. Both of the methods take an action (as a symbol), and an ActiveRecord class (an instance of which is the action's target).

### Being specific

You can be more specific about what a user can and cannot do:

```ruby
can :write, Article, owner_id: user.id
```

...allows a user to be able to `write` an `Article` as long as the user's `id` matches the `owner_id` of the Article.

```ruby
can :read, Article, :category => { :visible => true }
```

..allows a user to `read` an `Article` as long as the article's `category` has it's `visible` column set to `true`. Very cool.

```ruby
    class Ability
      include CanCan::Ability
 
      def initialize(user)
        can :read, Post
        can :create, Post
        unless user.nil? # guest
          # CanCan accepts a hash of conditions;
          # here, we're saying that the Post's user_id
          # needs to match the requesting User's id
          can :update, Post, { user_id: user.id }
        end
        if user.admin?
          can :manage, Post
        end
      end
    end
```

This is an example `Ability` class where a user can:

- `read` a `Post`

- `create` a `Post`

- `update` a `Post` as long as:
    - they are a `User` (not a guest)
    - the user's `id` matches the `Post`'s `user_id`

- `manage` a `Post` as long as:
    - the user is an `admin`

The action `manage` is a special CanCanCan action which means "any action". So, that last line means that, if a user's `admin` column is set to `true`, they can do anything they want with any `Post`.