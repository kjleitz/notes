# ORM notes

Assume the language is Ruby and the SQL engine is sqlite3 unless otherwise specified. Assume we have set up the database object like we have [here](#db_setup). Sometimes results of queries will be accessed like arrays and sometimes they will be accessed like hashes. If this is confusing, see [here](#results_as_hash). Sorry about that.

<a name="db_setup"></a>
## Setting up a database object

Often enough, you might want to set up a database constant in your `config/environment.rb` file so it looks something like this:

```ruby
require 'sqlite3'
require_relative '../lib/song.rb'
 
DB = {:conn => SQLite3::Database.new("db/music.db")}
```

Now you have `DB` set up as a hash containing the connection to the database. You can then access the database connection by using `DB[:conn]` wherever you'd like.

**_Assume we have done this for the remainder of the notes._**

<a name="results_as_hash"></a>
### Getting `SELECT` results as a hash

You get your rows back as an array, by default. Instead, you might want to get it back as a hash, with the columns as keys! You can accomplish this like so:

```ruby
DB[:conn].results_as_hash = true
```

Put that right in your `environment.rb` file and it'll return a hash every time! (I believe it's actually an array of hashes, but I'll check that out) (Yes) (Thanks) (No problem) (Stop talking to yourself) (Okay)

Looks like you can still access the elements of the hash with numbers, like an array, if you want. They seem to be present as keys in the hash, making them interchangeable. Don't quote me on that. But this doubles the size of your hash, doesn't it? Might make large datasets tricky to do this with.

## Executing SQL against the database

Execute arbitrary SQL like this:

```ruby
DB[:conn].execute("SELECT * FROM table WHERE id = ?;", some_id)

# Also consider doing this:

sql = <<~SQL
  SELECT name, age
  FROM cats
  WHERE age < ?;
SQL

cat_array = DB[:conn].execute(sql, 12)

```

<a name="interpolation"></a>
### Interpolation in the SQL string

The `#execute` method interpolates extra arguments where there are `?`s in the string. They're called "binding parameters".

You cannot use binding parameters to change the structure of a SQL statement (e.g. for table names, for column names, etc.). With that, you just have to do some _(careful, of course)_ interpolation.

## Getting the columns/info of a table

You can retrieve some info (like column names, their types, their deets, etc.) from a table by using the following:

```ruby
sql = "PRAGMA table_info(<table name>);"

DB[:conn].execute(sql)
# returns:
[{"cid"=>0,
  "name"=>"id",
  "type"=>"INTEGER",
  "notnull"=>0,
  "dflt_value"=>nil,
  "pk"=>1,
  0=>0,
  1=>"id",
  2=>"INTEGER",
  3=>0,
  4=>nil,
  5=>1},
 {"cid"=>1,
  "name"=>"name",
  "type"=>"TEXT",
  "notnull"=>0,
  "dflt_value"=>nil,
  "pk"=>0,
  0=>1,
  1=>"name",
  2=>"TEXT",
  3=>0,
  4=>nil,
  5=>0},
  # ...
  ]
```

## Patterns

### String interpolation

Do **not** interpolate stuff into SQL strings with `#{blah}`. That leaves you susceptible to SQL injection attacks. Use [binding parameters](#interpolation) and it will escape things properly for you.

### Saving objects to a table on initialization

It's probably bad to have objects save to a database as soon as they are initialized. Don't design that way. Use a `#save` method to do it manually after it's instantiated, if you want to. What if there comes a situation where you want to make objects but not save them? If that logic is inherent to its initialization, that's bad news.

To instantiate an object and add it at the same time, make a separate method called `.create` that just wraps `.new` and `#save`. Classic.

### Have an `id`, but don't do it yourself

Let the database handle setting `id`s. Don't handle it yourself. Get the `id` for the object from the database. For example, in a classic `#save` method, you might do this:

```ruby
  def save
  	if self.id
  	  self.update
  	else
      sql = <<~SQL
        INSERT INTO songs (name, album) 
        VALUES (?, ?)
      SQL
 
      DB[:conn].execute(sql, self.name, self.album)
 
      @id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs;")[0][0]
    end
  end
```

This checks to see if it has an `id` already (to avoid duplication), otherwise it saves it and grabs the `id` from the database. Apparently `last_insert_rowid()` is a valid function there, so use it wisely. And by wisely, I mean "look it up before you use it, because I don't know how it actually works beyond the obvious."

### Having an `.all` method that returns objects from the DB

If you set up a method like this:

```ruby
def self.new_from_db(row)
  new_song = self.new  # self.new is the same as running Song.new
  new_song.id = row[0]
  new_song.name =  row[1]
  new_song.length = row[2]
  new_song  # return the newly created instance
end
```

...to take a row returned from a `SELECT` on your table and turn it into an object, then you can also do this:

```ruby
class Song
  def self.all
    sql = <<-SQL
      SELECT *
      FROM songs
    SQL
 
    DB[:conn].execute(sql).map do |row|
      self.new_from_db(row)
    end
  end
end
```

...and return an array of _objects_ rather than rows.