# ERB notes

## Rendering a view

in the route method in your controller, use:

```ruby
get '/' do
  erb :index
end
```

...where `:index` refers to the `index.erb` file in your `views/` directory. Same with other views. Wrap them in quotes if you need to (`erb :"index"`).

## Embedding Ruby code

### Substitution tags (`<%= ... %>`)

Substitution tags look like this:

```html
<%= "Hello" + " World" %>
```

They evaluate the code inside and then display the return value.

### Scripting tags (`<% ... %>`)

Scripting tags look like this:

```html
<% if 1 == 2 %>
  <p>1 equals 2</p>
<% else %>
  <p>1 does not equal 2</p>
<% end %>
```

They evaluate the code inside but do not display anything themselves. You can do quite a bit inside them. Check it out:

```html
<ul>
<% squares = [1,2,4,8] %>
<% squares.each do |square| %>
  <li><%= square %></li>
<% end %>
</ul>
```

yields:

```html
<ul>
  <li>1</li>
  <li>2</li>
  <li>4</li>
  <li>8</li>
</ul>
```

### Accessing data outside the document

You can pass values in to the template and access them directly through instance `@variables`. You also have access to `params` from the request (`params[:some_key]`, remember?). Just remember that `params` is only available in the context of a single request, and `@variables` are only accessible within one route (can't access them in one route if they're defined in another).

## Inheriting layouts

Put your basic template in a file called `layout.erb`, and place a `yield` keyword where you want to insert other templates:

```html
<!doctype html>
<html>
  <head>
    <title>Cats</title>
    <link rel="stylesheet" href="/css/style.css">
  </head>
  <body>
 	<header>...</header>
    <div class="container">
      <h1>I love cats</h1>
      <img src="https://s3.amazonaws.com/after-school-assets/cat-typing.gif">
 
      <%= yield %>
      
    </div>
    <footer>...</footer>
    <script src="some_script.js"></script>
  </body>
</html>
```

Then, when you have a controller route:

```ruby
get '/cats' do
  erb :cats
end
```

and `cats.erb` looks like:

```html
<h2>OMG look at this cat...</h2>
<img src="http://cats.com/this_cat.jpg">
```

Then the rendered HTML looks like this:

```html
<!doctype html>
<html>
  <head>
    <title>Cats</title>
    <link rel="stylesheet" href="/css/style.css">
  </head>
  <body>
 	<header>...</header>
    <div class="container">
      <h1>I love cats</h1>
      <img src="https://s3.amazonaws.com/after-school-assets/cat-typing.gif">
 
      <h2>OMG look at this cat...</h2>
      <img src="http://cats.com/this_cat.jpg">
      
    </div>
    <footer>...</footer>
    <script src="some_script.js"></script>
  </body>
</html>
```

## Nested form data in `params`

So, if you have a form, you can access field values by their `name` as a key on the `params` hash, after the form is submitted:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="name">
  Student Grade: <input type="text" name="grade">
  Course Name: <input type="text" name="course_name">
  Course Topic: <input type="text" name="course_topic">
  <input type="submit">
</form>
```

...gives:

```ruby
params = {
  name: "Keegan",
  grade: "12",
  course_name: "Calculus",
  course_topic: "Math"
}
```

But that's a little ugly. We should have one set of information for the Student, and one set of information for the Course. You can achieve this with **nested** names:

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  Course Name: <input type="text" name="student[course][name]">
  Course Topic: <input type="text" name="student[course][topic]">
  <input type="submit">
</form>
```

...gives:

```ruby
params = {
  student: {
    name: "Keegan",
    grade: "12",
    course: {
      name: "Calculus",
      topic: "Math"
    }
  }
}
```

It's kind of like a reverse instantiation of a hash. You would normally _access_ the value `"Calculus"` by doing `student[:course][:name]`. So instead, you _set_ it like that (just without the symbols, and assuming you're operating on a bare hash called `students` instead of directly on `params`).

Okay, so what if we want to have multiple courses? What if we want the courses to be in an array of whatever size it needs to be based on how many fields are submitted in the form?

```html
<form action="/student" method="post">
  Student Name: <input type="text" name="student[name]">
  Student Grade: <input type="text" name="student[grade]">
  Course Name: <input type="text" name="student[courses][][name]">
  Course Topic: <input type="text" name="student[courses][][topic]">
  Course Name: <input type="text" name="student[courses][][name]">
  Course Topic: <input type="text" name="student[courses][][topic]">
  <input type="submit">
</form>
```

...gives:

```ruby
params = {
  student: {
    name: "Keegan",
    grade: "12",
    courses: [
      {
        name: "Calculus",
        topic: "Math"
      },
      {
        name: "Chemistry",
        topic: "Science"
      }
    ]
  }
}
```

So, the same basic concept is clear—you instantiate it like you access it. But, just like how you don't use symbols (just barewords) as your keys, you don't have to put numbers as indices. You just put a blank `[]`, and erb fills it in for you automatically, making a new element of the array every time the values repeat themselves, presumably.

This:

```ruby
student[courses][][name]
student[courses][][topic]
student[courses][][name]
student[courses][][topic]
```

...is:

```ruby
student[:courses][0][:name]
student[:courses][0][:topic]
student[:courses][1][:name]
student[:courses][1][:topic]
```

I know. It's weird. But it works, and you gotta just suck it up.