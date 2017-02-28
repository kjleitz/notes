# Rake notes

## The basics

Rake is used to automate tasks in Ruby.

Set up a Rakefile by making a file called (you guessed it!) `Rakefile` in the root directory of your app.

## Making tasks

Define tasks with `task` + name of task as a symbol + a block that contains the code we want to execute:

```ruby
task :hello do 
  puts "hello from Rake!"
end
```

### Descriptions

Describe your tasks with `desc` above your task:

```ruby
desc "outputs hello to the terminal"
task :hello do 
  puts "hello from Rake!"
end
```

Check out the descriptions with `rake -T`:

```
user@lappytoppy:~$ rake -T
rake hello  # outputs hello to the terminal
```
### Namespacing tasks

What if we wanna organize our tasks into related namespaces?

```ruby
namespace :greeting do 
desc 'outputs hello to the terminal'
  task :hello do 
    puts "hello from Rake!"
  end
 
  desc 'outputs hola to the terminal'
  task :hola do 
    puts "hola de Rake!"
  end
end
```

Execute the tasks like so:

```
user@lappytoppy:~$ rake greeting:hello
hello from Rake!
 
user@lappytoppy:~$ rake greeting:hola
hola de Rake!
```

### Require-ing files in a task

Say you wanna create a task to migrate/create your database like this:

```ruby
desc 'migrate changes to your database' 
namespace :db do 
  task :migrate => :environment do 
    Student.create_table
    ...
  end
end
```

You need access to `environment.rb` for the `Student` class. So, you have this bit:

```ruby
...
  task :migrate => :environment do
...
```

That creates a "task dependency". You create another task (called `:environment`) to `require` for you:

```ruby
task :environment do
  require_relative "config/environment"
end
```

This might seem roundabout (you could just `require` within the task itself) but I think it's so that you can set up different `require` environments for each task that can be easily reused. I'll have to check up on that.

## Executing tasks

In your terminal, you can type:

```
user@lappytoppy:~$ rake hello
hello from Rake!
```

See? It's that task from before! Whoa!

## Common patterns

### Making a task for dependency

Do this:

```ruby
desc 'sets up environment'
task :environment do
  require_relative "config/environment"
end
```

Then, say you wanna create a task to migrate/create your database, but you need to `require` a few things from your program in order to do that. Do this:

```ruby
...
task :migrate => :environment do 
  Student.create_table
end
```

Emphasis on this bit:

```ruby
...
  task :migrate => :environment do
...
```

That creates a "task dependency".

This might seem roundabout (you could just `require` within the task itself) but I think it's so that you can set up different `require` environments for each task that can be easily reused. I'll have to check up on that.

### Database migration

Doing common database setup and migration is often done in a `db:migrate` task:

```ruby
namespace :db do

  ...

  desc 'migrate changes to your database' 
  task :migrate => :environment do
    Student.create_table
    ...
  end
end
```

### Seeding a database

Setting up a database with dummy data is also a common task. Say you have a file `db/seeds.rb`:

```ruby
require_relative "../lib/student.rb"
 
Student.create(name: "Melissa", grade: "10th")
Student.create(name: "April", grade: "10th")
Student.create(name: "Luke", grade: "9th")
Student.create(name: "Devon", grade: "11th")
Student.create(name: "Sarah", grade: "10th")
```

Then you can set up a `db:seed` task:

```ruby
namespace :db do

  ...

  desc "seed the database with some dummy data"
  task :seed do
    require_relative "db/seeds.rb"
  end
end
```

### Load program and run a `pry` console

Sometimes you wanna play with your classes and test stuff out. Make a rake task for that:

```ruby
desc "pry console"
task :console => :environment do
  Pry.start
end
```