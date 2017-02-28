#Pry (ruby debug REPL) notes

---

**<u>Table of Contents</u>**

[toc]

---

Pry lets you put a breakpoint in your program and then continue in the terminal in a REPL.

##Basics

###require, then pry

```ruby
require 'pry'
```
needs to be in the file. Or, at least, it has to be in the file calling the program (a spec file, for example).

```ruby
def some_method
  blah = "Hello, I'm the first string"
  puts blah
  grrr = "Hi, second string, here."
  binding.pry
  puts grrr
end
```
use `binding.pry` to set a breakpoint.

```ruby
some_method   # => enters a REPL after printing the
              #    contents of 'blah'
```
it won't stop at definition time, it will only do it when the method or context or whatever is called.

Use `exit` to exit the REPL and allow the program to continue executing.

###The 'binding'

`binding` is an object that refers to the current context/scope, apparently.

