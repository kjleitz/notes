# Capybara notes

## Getting started

### Setup and configuration

First, we put the following in our `spec/spec_helper.rb` file:

```ruby
# Load RSpec and Capybara
require 'rspec'
require 'capybara/rspec'
require 'capybara/dsl'
 
# Configure RSpec
RSpec.configure do |config|
  # Mixin the Capybara functionality into Rspec
  config.include Capybara::DSL
  config.order = 'default'
end
 
# Define the application we're testing
def app
  # Load the application defined in config.ru
  Rack::Builder.parse_file('config.ru').first
end
 
# Configure Capybara to test against the application above.
Capybara.app = app
```

Here, we load up RSpec and Capybara. Then, configure RSpec to mix in Capybara functionality. Then, tell it what application we're testing (rack to parse `config.ru`). Finally, set Capybara's app-to-test to the application we defined just before.

### A basic test

A user visits the root route, they see "Welcome!" on the page somewhere:

```ruby
require 'spec_helper'

describe "GET '/' - Greeting Page" do
  it "welcomes the user" do
    visit '/'
    expect(page.body).to include("Welcome!")
  end
end
```

That's mostly plain RSpec. Capybara adds the `visit` and `page` methods.

## Main methods and objects

### The `#visit` method

Using `visit '/blah'` navigates Capybara's 'browser' to the route specified. It stashes the page you get back in an object called `page`.

### The `#page` method

The `#page` method exposes a `Capybara::Session` object which represents the browser page the user would be looking at (had they visited the route specified with `#visit`, or whatever the last action loaded).

### The `#fill_in` method

Calling `fill_in` fills forms:

```ruby
fill_in(:user_name, with: "kjleitz")
```

### The `#click_button` method

Calling `click_button` clicks buttons:

```ruby
fill_in(:user_name, with: "kjleitz")
click_button "Submit"
```

#### `#click_link`

Calling `click_link` probably clicks links and returns a new `page`, dunno.

#### Expectations

There are some common expectations you can have for `page`:

```ruby
expect(page.body).to include("Welcome to my site!")
# body of the page includes the words

expect(page).to have_text("Welcome to my site!")
# probably specifies plain text, bet the previous one
# includes all parts of the HTML doc

expect(page).to have_selector("form")
# there is a "form" HTML element present on the page
# I think this might also apply to CSS selectors generally

expect(page).to have_field(:user_name)
# page has a form input with a 'name' or 'id' attribute
# that matches 'user_name'
```