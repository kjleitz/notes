# Active Record notes

Check the [ORM notes](orm.md) for more general Object Relational Mapper stuff.

---

[toc]

---

## Establishing a connection

Make a connection to a database like so:

```ruby
ActiveRecord::Base.establish_connection(
  :adapter => "sqlite3",
  :database => "../db/dogs"
)
```

I'd put this in your `config/environment.rb` file. Then (also in that file, below it) you can create an object to represent that connection which you can access from anywhere:

```ruby
DB = ActiveRecord::Base.connection
```

And, below those, the `require_relative`s of your `lib/` files which contain your table model classes and whatever uses this stuff.

_Something is wrong with this section. The Learn.co material has you setting the first statement equal to `connection`, and then goes on to set `DB` equal to the connection (it seems). What's up with that? Then, to add to the confusion, in the `environment` file in the code-along to learn the stuff, it sets the first statement equal to `DB` and then sets the SECOND one to `DB`, too! What the fuck? So this is my best guess. Fix it if you find out otherwise._

**_Edit: The first statement is right._**

## Inheriting the base class

Do this:

```ruby
class Dog < ActiveRecord::Base
  ...
end
```

It will inherit a whole bunch of [nice stuff](http://guides.rubyonrails.org/active_record_basics.html) (all the stuff in your [ORM notes](orm.md) and more!).

## Executing SQL

Execute arbitary ruby against that database connection like so:

```ruby
sql = <<-SQL
  CREATE TABLE IF NOT EXISTS songs (
  id INTEGER PRIMARY KEY, 
  title TEXT, 
  length INTEGER
  )
SQL
 
ActiveRecord::Base.connection.execute(sql)
```

But you shouldn't really have to do that much, since inheriting the `ActiveRecord::Base` class will lend you a lot of helper methods to tack onto your table-modeling classes.

## Datatypes

ActiveRecord supports the following datatypes (may not be a complete list):

Datatype | Examples
--- | ---
boolean | true, false
integer | 2, -13, 485
string | "Halloween", "Boo!", (between 1-255 chars)
datetime | DateTime.now, DateTime.new(2014,10,31)
float | 2.234, 32.2124, -6.342
text | strings (between 1 and 2 ^ 32 - 1 chars)

## Useful methods

See [here](http://guides.rubyonrails.org/active_record_basics.html) for more methods and information. See [here](http://guides.rubyonrails.org/active_record_basics.html#crud-reading-and-writing-data) specifically for CRUD methods (**C**reate, **R**ead, **U**pdate, **D**elete)

### Create (insert) a record

Insert a new entry into your table:

```ruby
Student.create(name: "John Smith", grade: 9)
# INSERT INTO students (name, grade)
# VALUES ('John Smith', 9);
```

This is like doing `.new` and then `#save`.

### Get all records

Return all students:

```ruby
Student.all
```

### Get the first record

Return the first student:

```ruby
Student.first
```

### Find by id

Find a record by id:

```ruby
Student.find(3)
```

### Find by...

Find the first record by any attribute (column name):

```ruby
Student.find_by(name: "John Smith")
# SELECT * FROM artists WHERE name = 'John Smith' LIMIT 1
```

### Where conditional

Select from students where...:

```ruby
Student.where(name: "John Smith", grade: 10)
Student.where("grade > 10")
Student.where("grade > ?", some_grade)
```

### Ordering

Get all grade 10 students, sort by name descending:

```ruby
Student.where(grade: 10).order(name: :desc)
```

### Column names

Get the column names of your table:

```ruby
Student.column_names
```


### Attribute Accessors

You can get or set attributes of an object once you've retrieved it:

```ruby
john = Student.find_by(name: "John Smith")
john.name
#=> "John Smith"

john.name = "Sven"
john.name
#=> "Sven"
```

### Save changes to database

Save the changes you've made to an object in the table (as its record):

```ruby
john = Student.find_by(name: "John Smith")
john.name = "Sven"
john.save
```

### Update object

Make your changes and save them at the same time:

```ruby
d = Student.find_by(name: "David")
d.update(name: "Dave")
```

There's also `#update_all`, which updates all given records with the values supplied:

```ruby
Student.where(grade: 9).update_all(grade: 10)
```

### Delete a record

Delete an object from the database entirely:

```ruby
john = Student.find_by(name: "John Smith")
john.destroy
```

Or, `destroy_all` (like `update_all`):

```ruby
Student.where(grade: 12).destroy_all
```

### Aggregate functions

#### Sum

Sum all the values of a particular column:

```ruby
Student.sum(:grade)
```

#### Minimum

Get the minimum value of a particular column:

```ruby
Student.minimum(:grade)
```

#### The rest

They're pretty self-explanatory. Check 'em out [here](http://api.rubyonrails.org/classes/ActiveRecord/Calculations.html).

## Macros

### Making associations

There are ActiveRecord macros to take care of associations/relationships between models and other models/tables, like "belongs to," "has many," "has many through," etc.

#### "Belongs to"

A song belongs to an artist and to a genre (both other model classes):

```ruby
class Song < ActiveRecord::Base
  belongs_to :artist
  belongs_to :genre
end
```

You'll need the foreign key column to be on the table which `belongs_to` something else:

```ruby
class AddForeignKeysToSongs < ActiveRecord::Migration
  def change
    add_column :songs, :artist_id, :integer
    add_column :songs, :genre_id, :integer
  end
end
```

Woo! Magic.

#### "Has many"

An artist has many songs (another model class):

```ruby
class Artist < ActiveRecord::Base
  has_many :songs
end
```

Because our songs table has an `artist_id` column, and because we've used the `has_many` macro, an artist now has many songs.

**Just a note:**

ActiveRecord is also smart enough to know that when you set an attribute of `song_ids` on that artist, like a form passing an array of `song_ids` through `params` into an artist creation, it will create those connections:

```ruby
# This is what params looks like, coming from a form
# that POSTs the info for a new user-created artist:
# {
#   artist: {
#     name: Adele,
#     song_ids: [3, 6, 7]
#   }
# }

artist = Artist.create(params[:artist])
artist.songs.first.id
# => 3
``` 

Awesome!

#### "Has many...through"

An artist also has many genres, through its songs:

```ruby
class Artist < ActiveRecord::Base
  has_many :songs
  has_many :genres, through: :songs
end
```

A genre, similarly, looks like this:

```ruby
class Genre < ActiveRecord::Base
  has_many :songs
  has_many :artists, through: :songs
end
```

Since the songs table has an `artist_id` column and a `genre_id` column, it acts as a JOIN table for the two.

#### "Many to many" & JOIN tables

A JOIN table, though, is another topic. When you're setting up associations, you might have a specific JOIN table that is just, like, `user_id` and `item_id` (to use a different example). You might have `Item`s which belong to many `User`s, and `Users` which have many `Item`s, so you need a way to connect the two with backwards-and-forwards "has many...through" relationships. So you make the tables `users` and `items` as usual (without foreign keys) and a table `user_items` which keeps track of what belongs to what:

```ruby
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
    end
end
 
class CreateItems < ActiveRecord::Migration
  def change
    create_table :items do |t|
      t.string :name
      t.integer :price
    end
  end
end
 
class CreateUserItems < ActiveRecord::Migration
  def change 
    create_table :user_items do |t|
      t.integer :user_id
      t.integer :item_id
    end
  end
end
```

So, then, you make the appropriate relationships between them:

```ruby
class User < ActiveRecord::Base
  has_many :user_items
  has_many :items, through: :user_items
end
 
class Item < ActiveRecord::Base
  has_many :user_items
  has_many :users, through: :user_items
end
 
class UserItem < ActiveRecord::Base 
  belongs_to :user
  belongs_to :item
end
```

Nice. Recap:

- `UserItems` belongs to a `User` and an `Item`

- `User` has many `user_items` entries
	- `User` has many `Item`s _through_ `user_items`

- `Item` has many `user_items` entries
	- `Item` has many `User`s _through_ `user_items`

#### Using "custom" foreign id (this title could be better)

What if you have an AirBnB-type app, where:

- Listings belong to a host
- Reservations belong to a guest
- Reviews belong to a guest

We could create two tables, `hosts` and `guests` along with two corresponding models, but that makes things too complicated. Instead, let's have both "host" and "guest" be instances of `User`. How do we make those associations?

```ruby
class Listing
  belongs_to :host, class_name: "User"
end
```

Now the `listings` table has a `host_id` column. We can use it in the `has_many` association on `User`:

```ruby
class User
  has_many :listings, foreign_key: "host_id"
end
```

Otherwise, ActiveRecord would default to using `user_id` when looking it up by the foreign key.

You might have a join table in certain contexts where you need to access members through it and they need to be named something else:

```ruby
# user.rb
has_many :notes
has_many :viewers
has_many :readable, through: :viewers, source: :note
 
# note.rb
belongs_to :user
has_many :viewers
has_many :readers, through: :viewers, source: :user

# viewer.rb
belongs_to :user
belongs_to :note
```

In that example, `viewers` is a join table denoting which notes can be read by which user.

### Using associations

[This](https://www.youtube.com/watch?v=5dqPYRsQd10) is a great Avi video on ActiveRecord Associations and their macros.

[This is also](https://www.youtube.com/watch?v=l9JCzNN2Z2U) a GREAT video on all the usual ActiveRecord macros involved with associating objects.

The association macros give your models attributes that are accessible with standard getters and setters. Now, you can do:

```ruby
adele = Artist.new(name: "Adele")
hello = Song.new(name: "Hello")

hello.artist
#=> nil

hello.artist = adele

hello.artist.name
#=> "Adele"
```

But notice:

```ruby
adele.songs
#=> []
```

**Important:** The object that `belongs_to` another object is the **child**. The object that `has_many` objects is the **parent**.

- Telling the **child** who its **parent** is _does not_ let the parent know about its child
- Telling the **parent** who its **child** is lets the child know who its parent is, too

```ruby
rolling_in_the_deep = Song.new(name: "Rolling in the Deep")

adele.songs << rolling_in_the_deep

adele.songs.first.name
#=> "Rolling in the Deep"

rolling_in_the_deep.artist.name
#=> "Adele"
```

Woo!

You can also access your `has_many...through` attributes, but it seems like that only works if you have `create`d (or maybe just initialized and `save`d?) the record to the database:

```ruby
pop = Genre.create(name: "pop")

pop.songs << rolling_in_the_deep

rolling_in_the_deep.genre.name
#=> "pop"

pop.songs.first.name
#=> "Rolling in the Deep"

pop.artists.first.name
#=> "Adele"
```

Cool.

#### Shoveling can be inefficient

Turns out that this:

```ruby
pop.songs << rolling_in_the_deep
```

...returns all of the songs in `pop.songs`, putting all of that (if it's a big database) into memory, which can be very inefficient. It also automatically does the SQL query immediately. ActiveRecord gives us an alternative method called `#build` (as opposed to `<<`) to use on collections:

```ruby
some_song = kanye.songs.build(name: "Some Song")
#=> A song object named "Some Song", with Kanye's
#   artist_id automatically filled properly

some_song.save
#=> Performs the INSERT and doesn't have to load the
#   whole list into memory

# You could also do kanye.save; either way works.
```

There's also the `#build_artist` method, too... So, if you have a `belongs_to` relationship, for example, you can use `build_<association>` to build an artist to save on the song, in a similar fashion:

```ruby
some_song.build_artist(name: "Kanye")
some_song.save
```

You can go build and have it go straight to the database by saying `create` instead of `build`, like `#create_artist`.

**POINT IS,** just be careful about when you `<<` shovel stuff in. It can use a lot of memory handling what's returned to you. According to Avi, apparently we _usually_ use `#build`/`#build_<association>`, then manually `#save` when we want to, instead of shoveling and using `create` and whatever.

#### Objects may be cached and not fully reflect the database

If you've called `kanye.songs`, ActiveRecord performs the SQL query and then stores that in memory for a while, and (if I'm not mistaken) doesn't go back to the table to grab new values every time you call it. So, it can get "stale" and you have to do a `kanye.reload` or `some_song.reload` or whatever. Not super sure exactly when this happens, but you should be aware that it _does_, and the `#reload` method can refresh that object's associations.

### Making hashed passwords

You need to use the `bcrypt` gem for the following functionality.

Add a `password_digest` string column in your `users` table:

```ruby
Class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :username
      t.string :password_digest
    end
  end
end
```

Use the `has_secure_password` macro in the `User` model:

```ruby
class User << ActiveRecord::Base
  
  has_secure_password
  
end
```

Now, you can still access the `password` attribute on `User` as if it were a column (even though you have `password_digest` instead)!

```ruby
post "/sign_up" do
  user = User.new(:username => params[:username], :password => params[:password])
  redirect user.save ? "/success" : "/failure"
end
```

How awesome is that?

Then, you can use `user.authenticate` to check the plain text password against the `password_digest` on the `User`:

```ruby
post "/login" do
  user = User.find_by(:username => params[:username])
  if user && user.authenticate(params[:password])
    session[:user_id] = user.id
    redirect "/success"
  else
    redirect "/failure"
  end
end
```

### Validating models

You can use [ActiveRecord validations](http://guides.rubyonrails.org/active_record_validations.html) to enforce the presence of certain attributes:

```ruby
class Person < ActiveRecord::Base
  validates_presence_of :name
end
 
Person.create(name: "John Doe").valid? # => true
Person.create(name: nil).valid? # => false
```

The second `Person` will not be persisted to the database.

#### Presence

But I think that's the deprecated syntax. You might want to look [here](http://api.rubyonrails.org/classes/ActiveModel/Validations/ClassMethods.html#method-i-validate) for more info on validation. Luke from Flatiron showed me that in my sinatra application, instead of doing this:

```ruby
class User < ActiveRecord::Base
  validates_presence_of :username
  validates_presence_of :email
end
```

...I should do this:

```ruby
class User < ActiveRecord::Base
  validates :username, email, presence: true
end
```

Which is better.

#### Uniqueness

You can also validate **uniqueness** of a model's attributes. Say you don't want a user to be made with a username or email which have already been used:

```ruby
class User < ActiveRecord::Base
  validates :username, email, presence: true
  validates :username, email, uniqueness: true
end
```

Those might be able to be chained, too, but you'll have to see if this works:

```ruby
class User < ActiveRecord::Base
  validates :username, email, {
    presence: true,
    uniqueness: true
  }
  
  # or maybe just...
  
  validates :username, email, presence: true, uniqueness: true
  
  # only one way to find out!
end
```

#### Formatting

You can even validate the formatting of attributes! Instead of manually checking to make sure the `username` or `email` matches a regex, for example, you might do this:

```ruby
class User < ActiveRecord::Base

  validates :username, email, presence: true
  
  validates :username, email, uniqueness: true

  validates :username, format: {
    with: /\A\w{1,80}\z/,
    message: "may only contain letters, numbers, and underscores, and be less than 80 characters in length."
  }
  
  validates :email, format: {
    with: /\A([\w+\-].?)+@[a-z\d\-]+(\.[a-z]+)*\.[a-z]+\z/i,
    message: "must be valid"
  }

end
```

You can see here that you can also add custom `message`s to the model's `errors` in the event that the attribute does not validate.

### Creating custom scopes

Check [this](http://api.rubyonrails.org/classes/ActiveRecord/Scoping/Named/ClassMethods.html#method-i-scope) out. You can define custom scopes (essentially the same as defining a class method on the model, say, `.red` which calls `self.where(color: "red")`):

```ruby
class Shirt < ActiveRecord::Base
  scope :red, -> { where(color: 'red') }
  scope :dry_clean_only, -> { joins(:washing_instructions).where('washing_instructions.dry_clean_only = ?', true) }
end
```

Now you can do:

```ruby
Shirt.red
# => returns an ActiveRecord::Relation which is just
#    an ActiveRecord array of items which match, like
#    how doing Shirt.all returns one for all the shirts
```

### Callbacks

These are macros that allow you to run a method or some logic before or after some event happens. See [here](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html). For example:

```ruby
class User < ActiveRecord::Base
  has_one :post
  after_create :make_welcome_post
  
  private
    def make_welcome_post
      self.post = Post.create(content: "Welcome!")
    end
end
```

You can see how these might be useful.

## Scopes

Yoooooooo take a look at [this shit](http://api.rubyonrails.org/classes/ActiveRecord/Scoping/Default/ClassMethods.html).

That's _useful_, broseph.

## Migrations

Set up migrations so that you can easily and simply apply new database changes and revert them if you need to. Migrations are like version control for your database. Executed migrations are tracked by ActiveRecord so they aren't used twice and you don't have to keep track of changes manually.

USE MIGRATIONS TO CREATE YOUR TABLES. All table creation and alteration should be done through migrations, generally.

### Making a new migration

Make a folder `db/migrate/` and then put a file in there called something like `01_create_artists.rb` (name them in sequential order so that the migrations happen correctly, because they are run alphanumerically... Rails apparently does this automatically):

```ruby
class CreateArtists < ActiveRecord::Migration
  def up
    ...
  end
  
  def down
    ...
  end
end
```
**Important:** I realized that the `rake db:migrate` task takes the filename and assumes your class is TitleCase'd from that (being originally snake_case), to a tee. I get errors with `04_add_age_to_students` having a class name of `AddAgeToStudentz` but not `AddAgeToStudents`. Huh!

**Also important:** Apparently inheriting directly from `ActiveRecord::Migration` is deprecated. Check on that.

The `#up` method defines what to execute when the migration is run, and `#down` is for what to execute when it's rolled back ("do" and "undo"). More common for basic migrations is `#change`:

```ruby
class CreateArtists < ActiveRecord::Migration
  def change
    ...
  end
end
```

...which is just short for "do this/undo it on rollback".

#### THE EASY WAY!

Turns out you can set up a template migration file with:

```
user@lappytoppy:~$ rake db:create_migration NAME=create_artists
```

That'll populate a file (with a timestamped filename) with a class set up exactly like the last example.

### Writing a migration

We can execute database stuff with methods within that `#change` method:

```ruby
class CreateArtists < ActiveRecord::Migration
  def change
    create_table :artists do |t|
      ...
    end
  end
end
```

You can (in addition to `create_table`) do other methods like `remove_table`, `drop_table`, `rename_table`, `remove_column`, `add_column`, etc. See more methods [here](http://edgeguides.rubyonrails.org/active_record_migrations.html) ([here specifically](http://edgeguides.rubyonrails.org/active_record_migrations.html#writing-a-migration)).

We'll add columns to that created table like so:

```ruby
class CreateArtists < ActiveRecord::Migration
  def change
    create_table :artists do |t|
      t.string :name
      t.string :genre
      t.integer :age
      t.string :hometown
      t.timestamps
    end
  end
end
```

**Sidenote:** `t.timestamps` makes two new columns, `created_at` and `updated_at`, super useful if you want to query by timestamp instead of by `id` or something! Check out the docs on it [here](http://api.rubyonrails.org/classes/ActiveRecord/Timestamp.html). They aren't necessary, but I wanted to throw that in.

The `create_table` method takes a table name (as a symbol) as an argument, and a block (with the table passed as a parameter) where you can set up your columns. The columns are formed by calling the type as a method on the table, and supplying it a column name (as a symbol). It will automatically make an `id INTEGER PRIMARY KEY` column for you which auto-increments like normal.

I don't know if this is calling `create_table` on the `Artist` class, or what. I don't think it is. I'm not sure if there is such a method, actually, but I assume there is. Check on that.

Another quick example: Say you want to add a column to that table. You'd want to create a new file, `db/migrate/02_add_gender_to_artists.rb`, and do the following:

```ruby
class AddGenderToArtists < ActiveRecord::Migration
  def change
    add_column :artists, :gender, :string
  end
end
```

Easy peasy! `add_column` then the table name, the new column name, and the new column type (all as symbols).

### Running a migration

The simplest way to run a migration that we have set up is through a rake task that the `activerecord` gem has prepared for us. In your rakefile, use this line:

```ruby
require 'sinatra/activerecord/rake'
```

(Probably different if you're using rails? ...considering it has `sinatra` in the path. Not really sure, not there yet).

This should also be in the `environment` file:

```ruby
ActiveRecord::Base.establish_connection(
  :adapter => "sqlite3",
  :database => "db/artists.sqlite"
)
```

...so that we have made a connection to the `artists` database.

You can see what rake tasks ActiveRecord has given us by running `rake -T`. In order to execute all your migration files up to this point, do this:

```
user@lappytoppy:~$ rake db:migrate
```

...and your database should be all ready to go and migrated!

### Rolling back a migration

You can roll back the last migration you made with:

```
user@lappytoppy:~$ rake db:rollback
```

