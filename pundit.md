# Pundit notes

Pundit allows you to organize your authorization code so you don't end up with an overly complex CanCanCan `Ability` class, or controllers overloaded with authorization checks.

## Basic setup

Add this line to your gemfile:

```ruby
gem 'pundit'
```

Then, `include` the module into your main `ApplicationController`:

```ruby
class ApplicationController < ActionController::Base
  include Pundit
  # ...
end
```

Now, install Pundit into your application (sorta like Devise):

```
$ rails g pundit:install
```

## Policy classes

You manage your authorization rules in Pundit `Policy` classes. They're named like `<model name>Policy`, e.g. `PostPolicy` or `UserPolicy`. They are kept in `app/policies`:

```ruby
# app/policies/post_policy.rb

class PostPolicy < ApplicationPolicy
  def update?
    user.admin? || user.moderator? || record.try(:user) == user
  end
end
```

In that example, users who are admins or moderators can update a post, and if not, then only if the user is the post's owner.

The `record` object is the instance of the model it's checking the policy for. You can change this to be a little more colloquial:

```ruby
class PostPolicy < ApplicationPolicy
  attr_reader :post
 
  def initialize(user, post)
    super(user, post)
    @post = record
  end
 
  def update?
    user.admin? || user.moderator? || post.try(:user) == user
  end
end
```

...but I think `record` is good enough, and maybe you can take advantage of the abstraction for related code.

## Using a `Policy` in the controller

You can use the policy to authorize in the controller like so:

```ruby
class PostsController < ApplicationController
  def update
    @post = Post.find(params[:id])
    authorize @post
  end
end
```

The `#authorize` method becomes available to the controller when you `include Pundit` in the `ApplicationController`. The method calls the object's `Policy` method named the same as the action you are calling it from in the controller, plus a `?`. So, for that example, it basically does this:

```ruby
PostPolicy.new(current_user, @post).update?
```

If you want to specify the action yourself, you can construct a new `PostPolicy` object with `current_user` and `@post`, then call whatever `<action>?` on it. Or, you can use the nice helper method `#policy` to construct it more easily, and then call the method yourself:

```ruby
policy(@post)
# which is equivalent to:
PostPolicy.new(current_user, @post)
```

For example, you can use it in your views like so:

```html
<% if policy(@post).update? %>
  <%= link_to "Edit post", edit_post_path(@post) %>
<% end %>
```

Nice.

## Scopes

Say we want to show a user what posts they have access to... for example, there are drafts, and there are published posts. Only owners of drafts and admin are allowed to see unpublished (draft) posts. 

We can achieve this with scopes! Define a class named `Scope`, which inherits from `Scope` (weird, I know), inside of your model's `Policy` class:

```ruby
class PostPolicy < ApplicationPolicy
  class Scope < Scope
    def resolve
      if user.admin?
        scope.all
      else
        scope.where(:published => true)
      end
    end
  end
  # ...
end
```

That `Scope` class initializes with a user and an `ActiveRecord::Relation`, a.k.a. what you would get from a `where` query. So, for example, you can use it in your view so a user can see all the posts that they are allowed to see, using the helper method `policy_scope`:

```html
<% policy_scope(@user.posts).each do |post| %>
  <p><%= link_to post.title, post_path(post) %></p>
<% end %>
```

## Unit testing

You can test policies pretty easily with Pundit:

```ruby
class PostPolicyTest
  test "users can't update others posts" do
    amethyst = users(:amethyst)
    post = posts(:garnet_private)
    expect(post.user).not_to eq(amethyst)
    expect(PostPolicy.new(amethyst, post).update?).to be false
  end
end
```
