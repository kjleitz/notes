# Ruby notes

---

**<u>Table of Contents</u>**

[toc]

---


## General

### Pass by value

Variables are pass-by-value, not by reference... WHOA THERE, full stop! Technically this is true. But ugh. Look. This happens:

```ruby
foo = "hello"
bar = foo
foo          # => "hello"
bar          # => "hello"
bar << "ha"
foo          # => "helloha"
bar          # => "helloha"
```

...but then this ALSO happens:

```ruby
foo = "hello"
bar = foo
foo          # => "hello"
bar          # => "hello"
bar += "ha"
foo          # => "hello"
bar          # => "helloha"
```

I mean, come on. So, as far as I understand it, the variable holds a reference to the string _object_. Two identical string primitives are not the same object, by the way. So the variable `foo` holds a reference to the string object, then `bar` is set equal to that same reference to the object. When you _shovel_ in another string, you shovel it directly onto the string in the object `bar` references (the same object `foo` references). When you use the `+=` operator, you _assign_ bar to a new string object created by concatenation of two string objects, I think. You're basically saying:

```
bar = String((the string in the object referenced by bar) + (some new string))
```

That's at least a little interesting, though, that the shovel operator works that way. Makes sense why it's "faster".


## Methods

### Implicit returns

A method returns the return value of the last statement in the method body, unless something is explicitly returned. Apparently the former is good style. I think that's ridiculous. That's going to result in less-readable code, and if someone else comes along and modifies the method they might break a return value without realizing it was important somewhere else. If the only 'con' to using an explicit return is that you have to type it, and the only benefit to using an implicit return is that it's less to type, I'm going to be explicitly returning forever.

### Default arguments

Blah blah just add 'em as `def foo(bar=4, xyz="poo")`


<a name="arg_hash"></a>
### Named arguments/Argument hash

You can pass keyword arguments (basically an exposed hash) to a method to name them.

```ruby
def foo(bar:, xyz:)
  # named args, can put 'em in whatever order at call time
  "#{bar} #{xyz}, please."
end

def foo(bar: 4, xyz: "poo")
  # default values for the named args
  "#{bar} #{xyz}, please."
end

foo(bar: 10, xyz: "salami")
# => "10 salami, please."

foo(xyz: "salami", bar: 10)
# => "10 salami, please."
```

You can also _literally_ pass it a hash when you call it, with the same keywords, and it'll work as expected:

```ruby
arg_hash = {xyz: "salami", bar: 10}
foo(arg_hash)
# => "10 salami, please."
```

This is actually _really_ common in web APIs, so get used to it!

There's also a bit of metaprogramming you can do if you want your object to accommodate all the keys of the hash as arguments even if they're different what you added explicitly when defining them. So, for example, say you want to get a user's info from Twitter, and it sends you a hash of their info. Maybe Twitter's API won't give you some data you were expecting. You didn't know about these changes, you haven't changed your method, and you are going to get some errors. Here's what you do instead from the get-go:

```ruby
twitter_user = {
  username: "kjleitz",
  name: "Keegan",
  location: "Boston"
}

class User
  attr_accessor :username, :name, :age, :address, :location
  def initialize(user_hash)
    user_hash.each do |key, value|
      self.send("#{key}=", value)
    end
  end
end

keegan = User.new(twitter_user)
keegan.location  # => "Boston"
keegan.age       # => nil
```

So, then, as the argument hash might change, you just have to change your `attr_accessor`s. Your `initialize` method is flexible and adequate!


## Blocks

### `do...end` vs. `{}`

Braces have a higher precedence than `do` blocks, but they essentially do the same thing. Note that in the following code...

```ruby
1.upto 3 do |x|
  puts x
end
```

...seems the same as...

```ruby
1.upto 3 { |x| puts x }
```

...but the latter gives a `SyntaxError: compile error`, because of the precedence. So, you have to use parentheses instead...

```ruby
1.upto(3) { |x| puts x }
```

...I should look up why this is exactly. Apparently doing the first way (and the third way) will pass the block as an additional parameter to the first method. But in the second way, because the braces have higher precedence order, the block is passed as an argument to the second "method" which is 3, and then the results of that are passed to the first method. So it metaphorically looks like this...

```ruby
1.upto(3({ |x| puts x }))
```

...instead of...

```ruby
1.upto(3)({ |x| puts x })
```

[See here.](http://stackoverflow.com/questions/2122380/using-do-block-vs-brackets)

## Arrays (slash other enumerables)

### Array methods

`Array#detect` is the same as `Array#find`

`Array#select`/`#reject` are great basic filters for arrays; you don't use these enough.

Also, `Array#any?`/`#all?`/`#none?`/`#include?` are great boolean enumerators, use them too.

`#flatten` can give you a flat, non-nested array, that's nice if it comes up.

### Iterating over arrays

Neat trick if you're, say, `#map`ping an array and you want the index as well:

```ruby
some_array = ['red', 'green', 'blue']

some_array.map {|color| color + "1"}
# => ['red1', 'green1', 'blue1']

some_array.map.with_index {|color, index| color + "#{index+1}"}
# => ['red1', 'green2', 'blue3']
```

You can tack on the `#with_index` method to any `Enumerable` (which is what's returned by `#map` and related methods if no block is given).

### Using `inject`

This is cool. Works on any enumerator (I think). The `.inject` method allows you to pass it a block with two parameters (as an "accumulator value" (_memo_) and as the current element, respectively), and that first parameter is given the return value of the block for the next cycle. You can specify an initial value for _memo_, or leave it blank and it will just take the first element of the collection you are iterating over. (That I do not get... shouldn't it double-count the first element if that were the case?)

```ruby
array = [1, 2, 3, 4]

array.inject { |sum, n| sum + n }       # => 10
array.inject(3) { |sum, n| sum + n }    # => 13
array.inject { |prod, n| prod * n }     # => 24
array.inject(2) { |prod, n| prod * n }  # => 48


array2 = [cat, horse, alligator, emu]

longest = array.inject do |memo, word|
  memo.length < word.length ? word : memo
end

longest  # => "alligator"
```

### Deleting values

See the [`#delete_if` section](#delete_if) in Hashes, same thing applies.

### Proc invocation shorthand (`#map(&:foo)`)

Want a method to be called on each member of an array:

```ruby
abc = ["a", "b", "c"]

abc.map { |letter| letter.upcase }
# => ["A", "B", "C"]

abc.map(&:upcase)
# => ["A", "B", "C"]
```

How cool is that? The `&` turns the method `#upcase` (as a symbol) into a proc, and applies the proc to every element in the array. You can also define your own proc and use it (without needing to specify it as a symbol):

```ruby
add_123 = Proc.new { |letter| letter + "123" }

abc.map(&add_123)
# => ["a123", "b123", "c123"]
```

You can do it for methods which would take an argument, too:

```ruby
abc = ["a", "b", "c"]

abc.each { |letter| puts letter }
# => "a"
# => "b"
# => "c"

abc.each(&method(:puts))
# => "a"
# => "b"
# => "c"
```

## Hashes

### Symbols as keys

If you use symbols for your keys, you can avoid using the hash rocket notation (which is so ANNOOOOYING to write), a.k.a.:

```ruby
my_hash = {:some_key => "value1", :another_key => "value2"}
```

...and instead you can write it like this:

```ruby
my_hash = {some_key: "value1", another_key: "value2"}
```

### Useful basic methods

- `#keys` will give you an array of all the keys in a hash
- `#values` will give you an array of all the values in a hash
- `#min` will return a two-member array with the key/value pair whose **key** is the lowest (not its value)

### Iterating over hashes

You can use `#each`, and it will yield the key and the value as a pair into the block:

```ruby
my_hash = {key1: "val1", key2: "val2"}
my_hash.each do |key, value|
  puts "#{key}: #{value}"
end
# Prints out:
#  key1: val1
#  key2: val2
```

### Using `#map`/`#collect` on hashes

`Hash#map` returns an _array_, not another hash. Just so you know.


<a name="delete_if"></a>
### Deleting values from hashes

This is useful: the `#delete_if` method. Iterates over the hash and deletes key/value pairs for which the condition provided in the block returns true:

```ruby
my_hash.delete_if do |key, value|
  key == :key_to_be_removed
end
```


## Yielding 'n stuff

### `yield` and enumerables

[good Avi lecture on yield and enumerables](https://www.youtube.com/watch?v=t2A6xPbh0I8)

Basically, if you want a method to take a block, just use the `yield` keyword. At that point in the method, it will run the code in the block, and then it will continue at the point it left off.

To pass a parameter to the block (as in, a `|piped|` parameter given to the block), just give it as an argument to the yield keyword. For example:

```ruby
def some_method(num)
  puts "Starting method..."
  yield num
  puts "All done!"
end

some_method(2) do |number|
  puts "I was given #{number} as an argument."
end

# Outputs:
#
# Starting method...
# I was given 2 as an argument.
# All done!
```

If you want the block to be optional, you can always use the `#block_given?` boolean to check if a block was given. Like this:

```ruby
def for_every_element(array)
  if !block_given?
    puts "Hey! No block was given!"
    return
  end
  array.length.times do |i|
    yield array[i]
  end
  array
end

for_every_element([1, 2, 3]) do |i|
  puts "number #{i}"
end
# prints "number 1\nnumber 2\nnumber 3"

hello_t([1, 2, 3])
# prints "Hey! No block was given!"
```

The return value of the block is returned by yield, too, so that's something you can also use:

```ruby
def mappish(array)
  i = 0
  mapped = []
  while i < array.length
    mapped << yield(array[i])
    i += 1
  end
  mapped
end
```

## Philosophy

### Single responsibility principle

Single Responsibility Principle - every method should do just one thing, have responsibility over a single functionality, and provide service narrowly aligned with that functionality. Lots of different methods building up a program make it much easier to build, test, and debug.

### API design and Test Driven Development

I like the fact that test-driven development kinda forces you to think about your program's API. You write it out the way you would use it (`author.stories.include?("To Kill A Mockingbird")`) and then design the code around that as you write.

### Versioning

This section is taken from [this Learn lesson on Gems and Bundler](https://learn.co/tracks/full-stack-web-development/object-oriented-ruby/scraping/gems-and-bundler):

All gems go through several different series of updates: a major version change or a minor version change. 

- A major version change is reflected by the first number (reading from left to right). Major version changes don't have to be backwards compatible. This means that if your app is built using version 1, and the gem updates to version 2, the new version can potentially break your app.

- A minor version change is reflected by the number after the first decimal point. All minor version changes have to be backwards compatible. This means that while version 1.2 has more functionality than version 1.0, all the features in 1.0 are supported in 1.2.

- The number after the second decimal point reflects a patch, which is a change to a gem to fix a bug but not introduce new functionality.

- 1.2.3 means major version 1, minor version 2, and a patch version 3.


## Classes

### When should I use `self`?

Pretty much only use `self` in these three scenarios:

- when setting/getting instance attributes inside a class definition
- to denote a method within the class definition as a class method
- to reference the calling object within an instance method definition

Taken from [here](https://hackhands.com/three-golden-rules-understand-self-ruby/).


### Order and grouping conventions

```ruby
class Person
  # extend and include go first
  extend SomeModule
  include AnotherModule

  # inner classes
  CustomErrorKlass = Class.new(StandardError)

  # constants are next
  SOME_CONSTANT = 20

  # afterwards we have attribute macros
  attr_reader :name

  # followed by other macros (if any)
  validates :name

  # public class methods are next in line
  def self.some_method
  end

  # initialization goes between class methods and other instance methods
  def initialize
  end

  # followed by other public instance methods
  def some_method
  end

  # protected and private methods are grouped near the end
  protected

  def some_protected_method
  end

  private

  def some_private_method
  end
end
```

### Class methods vs. instance methods

If you define a method in a class normally, it can only be called with instances of the class, not by the class itself (or globally, obviously):

```ruby
class Dog
  def bark
    puts "woof!"
  end
end

fido = Dog.new

fido.bark      # => outputs "woof!"
Dog.bark       # => undefined method
bark           # => undefined method
```

But if you define a class method, it will be able to be called from the class (but not from an instance of the class). It's defined by making an instance method on the `self` object--which is the class itself! Crazy. But yeah, a class is also an object, remember. So if you use the `self` keyword in the scope of the class definition (as opposed to inside of an instance method), it refers to the class as an object, not to the object initialized. Check it out:

```ruby
class Dog
  def self.bark
    puts "woof!"
  end
end

fido = Dog.new

fido.bark      # => undefined method
Dog.bark       # => outputs "woof!"
bark           # => undefined method
```


### Class variables vs. instance variables

So, instance variables are made like `@this`, and class variables are made like `@@this`. 

### Private vs. public methods

Pretty much all your normally defined methods and class methods are public (except `initialize`, that's private). You can make private methods by putting the keyword `private` above them. Then, any methods under that one use of the keyword `private` are private methods. You don't have to write private each time, it just goes ahead and blankets the rest of the class, I guess.

### Inheriting from other classes

You can inherit from another class with the following syntax:

```ruby
class Car < Vehicle
  ...
end
```

### Modules

You can use a module to lend a class a group of methods. Modules are different from superclasses because [insert reason here]. You can declare a module by doing:

```ruby
module Blah
  def babble
    "blah"
  end
end
```

And then in a class where you want to inherit the methods from that module as _instance methods_, use the `include` keyword:

```ruby
class Speech
  include Blah
end
```

If you want to inherit the methods as _class methods_, use the `extend` keyword:

```ruby
class Speech
  extend Blah
end
```

But having one separate module for instance methods and one for class methods is bad practice and less readable (which is for which purpose?). So you can use nested module namespacing to make it more clear:

```ruby
module FancyDance
  module InstanceMethods
  
    def twirl
      "I'm twirling!"
    end

    def take_a_bow
      "Thank you, thank you. It was a pleasure to dance for you all."
    end
  end
 
  module ClassMethods
    
    def metadata
      "This class produces objects that love to dance."
    end
  end
end
```

Then to use them you'd do:

```ruby
class Dancer
  extend FancyDance::ClassMethods
  include FancyDance::InstanceMethods
end
```

### Super

You can use the keyword `super` to pull in behavior from the parent class's/module's method of the same name at that point in execution, then continue with the rest of the stuff you want it to do.

```ruby
module Animal
  def initialize
    self.name = "Unnamed animal"
  end
end

class Cat
  include Animal

  attr_accessor :name, :gender

  def initialize
    super
    @gender = :female
  end
end

kitten = Cat.new

kitten.name    # => "Unnamed animal"
kitten.gender  # => :female
```

Now, how does this affect modules which might lend the same method names? Might want to look that up.


## Objects

### Freezing and 'frozen' objects

Something really cool: all objects can be "frozen". Have a property `author.stories` that's an array, but you don't want people to be able to `<<` things in, you only want it accessible via `author.stories.add_story(story)`?

**_Freeze it!_**

In your class definition, you can have the following:

```ruby
...
  def stories
    @stories.dup.freeze
  end

  def add_story(story)
    @stories << story if story.is_a?(Story)
  end
...
```

And that way, when someone tries to do `author.stories << some_story` they can't--an error is raised since the getter only exposes a frozen duplicate of the `@stories` instance variable.

It should be noted that there is no way to unfreeze a frozen object.

### Object relationships

Good stuff on `has_many` and `belongs_to` relationships between objects, shown [here in this Avi lecture](https://youtu.be/iYcQ693LXck). 


## Conditionals


### `unless`

```ruby
age = 10
unless age > 5
  puts "No school!"
else
  puts "Go to school!"
end
# ...this prints "Go to school"
```

### The `case` statement

Case statements, switch statements, whatever. Use `case` keyword, something you want to test the value of, and a series of `when` statements giving the matching value possibilities. Use `else` as the default action. Formed like this:

```ruby
name = "The Mad Hatter"
case name 
when "Alice"
  puts "Hello, Alice!"
when "The White Rabbit"
  puts "Don't be late, White Rabbit"
when "The Mad Hatter"
  puts "Welcome to the tea party, Mad Hatter"
when "The Queen of Hearts"
  puts "Please don't chop off my head!"
else 
  puts "Whoooo are you?"
end
```

But it barely has anything to do with **equality**. It uses `===` under the hood, which is the [_case subsumption_](http://stackoverflow.com/questions/4467538/what-does-the-operator-do-in-ruby) operator. See the [=== section](#===_operator) for more info.

So this:

```ruby
case foo
  when bar
    puts "hello"
  when xyz
    puts "goodbye"
end
```

...is this:

```ruby
if bar === foo
  puts "hello"
elsif xyz === foo
  puts "goodbye"
end
```

Oh, you can also do this:

```ruby
grade = 95
case grade
  when 90..100 then "A" 
  when 80..90 then "B"
  when 70..80 then "C"
  when 60..70 then "D"
  when 0..60 then "F"
end
```

## Operators

<a name="===_operator"></a>
### The `===` operator

It (sort of, depends on context) generally asks if the right-hand operand is described by the left hand operand. It's not symmetric, or transitive, either. For example:

```ruby
(1..8) === 4     # <= true
Numeric === 4    # <= true
Integer === 4    # <= true
String === 4     # <= false
String === "hi"  # <= true
4 === 4          # <= true
5 === 4          # <= false
[3, 4, 5] === 4  # <= false
```

So it's not necessarily _inclusion in a set_, it's more of, like, if you ask `foo === bar` then it would ask if `foo` is a container that `bar` would fit in, sorta. It changes based on the context of what is being compared to what, but it often coincides with the `#is_a?` method.


### Using boolean operators

`||` doesn't just return true or false... it returns the first operand if it is truthy, otherwise it returns the second operand.

```ruby
 true || true  # => true
    7 || 10    # => 7
false || 5     # => 5
false || nil   # => nil
```

`&&` doesn't just return true or false... it returns the first operand if it is false, otherwise it returns the second operand.

```ruby
 true && true  # => true
    7 && 10    # => 10
false && 5     # => false
    4 && nil   # => nil
```


### Using `!` to modify something in-place

Apparently it's ruby convention to use a `!` at the end of a method (like using `?` for booleans) to have it modify the object in-place. For example:

```ruby
messed_array = [2, 1, 3]
sorted_array = messed_array.sort
sorted_array   # => [1, 2, 3]
messed_array   # => [2, 1, 3]
```

Whereas:

```ruby
messed_array = [2, 1, 3]
sorted_array = messed_array.sort!
sorted_array   # => [1, 2, 3]
messed_array   # => [1, 2, 3]
```

## Not-so-obvious tricks for common situations

### Combined comparison operator: `<=>`

```ruby
6 <=> 5 = 1
5 <=> 5 = 0
5 <=> 6 = -1
```
...whaaaAAAaaAAaaAAAt?? Weird comparison operator. Super useful!

### Multi-line strings with[out] indentation (HEREDOC syntax)

You can use a HEREDOC to make a multi-lined string:

```ruby
class Subscription
  def warning_message
    <<-BLAH
      Subscription expiring soon!
      Your free trial will expire in #{days_until_expiration} days.
      Please update your billing information.
    BLAH
  end
end
```

What you put as `BLAH` doesn't matter. Yay! 

However, that will preserve the indentation you have there (6 spaces before each line).

To strip leading whitespace (normalizes all of the lines to the least-indented line), use a twiddle (`~`) instead of a dash (`-`) after the less-thans (`<<`), like this:

```ruby
class Subscription
  def warning_message
    <<~CRAP
      Subscription expiring soon!
      Your free trial will expire in #{days_until_expiration} days.
      Please update your billing information.
    CRAP
  end
end
```

So, `<<-SOMETHING...SOMETHING` for preserving that whitespace (who would want that?) and `<<~SOMETHING...SOMETHING` to left-strip them to the least-indented line (like you'd expect).

Keep in mind that I _think_ this was intrduced in Ruby 2.3.0, so versions below that might not work.

## REPLs

### irb
just use `irb`, no notes needed.

### pry

`pry` is a bit more useful. See notes [here](pry.md).


## Macros and metaprogramming

### Getters and setters

Use attribute writers and readers to make getter and setter methods more efficiently.

```ruby
class Person
  attr_writer :name
  attr_reader :name
end
```

...is the same as:

```ruby
class Person

  def name=(name)
    @name = name
  end
  
  def name
    @name
  end
end
```

...but even better is THIS:

```ruby
class Person
  attr_accessor :name
end
```

The `attr_accessor`. The **attribute accessor**! Woo!

[Good video by Avi on metaprogramming and object-object relationships.](https://www.youtube.com/watch?v=ab11lJJKm8M)

<a name="arg_hash_meta"></a>
### Argument hash metaprogramming

See the earlier section on ["Named arguments/Argument hash"](#arg_hash).

### The `send` method

`my_object.send(:method, argument)` is the same as `my_object.method(argument)`, but it allows you to abstract the method name/argument. If you pass `send` a string as the first argument, it converts it to a symbol. For a good example, see the [previous section](#arg_hash_meta). Also useful is the [ruby documentation](http://ruby-doc.org/core-2.3.1/Object.html#method-i-send).


## Errors

### Custom Errors

Declare a new error class by inheriting from some `Exception` class. I'd say just go with `StandardError` as the superclass, it's pretty much always fine. Basically this:

```ruby
class CustomError < StandardError
  def message
    "You did something wrong!"
  end
end

raise CustomError
```

### Handling errors (`rescue`)

You can handle errors like this:

```ruby
begin
  raise CustomError    # or just code that raises an error
  puts "I'm never gonna get puts'd, am I?"
rescue CustomError => error  # <-- can simply be 'rescue'
  puts error.message         #     or 'rescue CustomError'
ensure
  puts "I am definitely getting puts'd, regardless."
end
```

That way, it doesn't stop execution, it just displays the message and continues. That's the general pattern for it, at least.

Technically, all you need is the `rescue` keyword, the block isn't necessary if you just wanna catch all errors.


## Files and file structure


### Shebang line

shebang line for ruby: 

```
#!/usr/bin/env ruby
```

### Config/environment/requirements file

If you're going to need to require a bunch of files in a bunch of other files, why not reference a single file that imports the rest, that way you don't have to keep every file littered with `require`s (which seems hard to maintain)? I don't know if it's specifically good practice to do this everywhere, but it's something you definitely don't do enough.

In a structure like the following:

```
my_app/
  |-- bin/
  |    +-- app
  |-- concerns/
  |    +-- modules.rb
  |-- config/
  |    +-- environment.rb
  |-- db/
  |    +-- books.db
  |    +-- schema_migration.sql
  |-- fixtures/
  |    +-- example_data
  |-- lib/
  |    |-- author.rb
  |    |-- category.rb
  |    +-- story.rb
  +-- spec/
       |-- author_spec.rb
       |-- category_spec.rb
       |-- spec_helper.rb
       +-- story_spec.rb
```

Then `my_app/config/environment.rb` might look something like this:

```ruby
require_relative '../lib/author'
require_relative '../lib/category'
require_relative '../lib/story'
```

And then you can `require_relative '../config/environment'` wherever you need it. You can set up objects or whatever environment you need the program to run in/on.

- `bin/` is generally for the files that run your program.

- `concerns/` is for all your modules. Should be namespaced like this: `Concerns::ModuleName`

- `config/` is for setting up objects or requiring gems that the rest of your program can require in one place.

- `db/` is for your databases and `.sql` files, etc.

- `fixtures/` is for data that your specs might use to test your program.

- `lib/` is for the meat of your program.

- `spec/` is for specs/tests for your program.

#### Databases

Often enough, you might want to set up a database constant in your `config/environment.rb` file so it looks something like this:

```ruby
require 'sqlite3'
require_relative '../lib/song.rb'
 
DB = {:conn => SQLite3::Database.new("db/music.db")}
```

Then, you can access the database connection by using `DB[:conn]` wherever you'd like.

### Gemfile

#### The basic gemfile

Here is an example gemfile:

```ruby
source "https://rubygems.org" do
  gem "sinatra", "~> 1.4"
  
  group :development do
    gem "pry"
  end
end

gem "awesome_print", :git => "https://github.com/awesome-print/awesome_print"
```

That loads `sinatra` as a dependency that you can require, and `pry` if you're in the `:development` environment (`:development`, `:test`, and `:production` are the standard environments).

You can also do this:

```ruby
gem "pry", :group => "development"
```

which is called the hash syntax.

#### Dependency versions

- Need the Sinatra gem for your project? Add `gem 'sinatra'` to your Gemfile.
- Need the Sinatra gem, but at version 1.4.5? Add `gem 'sinatra', '1.4.5'` to your Gemfile.
- Need the Sinatra gem at a version higher than 1.4, but less than 1.5? Add `gem 'sinatra', '~> 1.4.0'`
- Need the Sinatra gem where it's at or higher than 1.5.3, and can have any higher minor version (but not a higher major version)? Add `gem 'sinatra', '~> 1.5', '>= 1.5.3'`

The symbol in `'~> 1.5'` means higher than that minor version, but within that major version. You can also (clearly) double up on conditions.


## Tests, specs, and test-driven-development

[Good Avi lecture, does some decent description of rspec. Long, but worth watching.](https://youtu.be/iYcQ693LXck)