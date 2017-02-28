# Nokogiri notes

---

[toc]

---

## The basics

### Accessing an HTML document

Use the `open-uri` gem for requesting a page, and obviously also require nokogiri:

```ruby
require "open-uri"
require "nokogiri"
```

Grab the html doc with `#open` and parse it with the `HTML` nokogiri method:

```ruby
html = open("http://www.google.com")
doc = Nokogiri::HTML(html)
```

## Navigating the tree

You can use CSS selectors to grab nodes:

```ruby
user_div = doc.css("#navigation .user-page div")
puts user_div.text  # puts the inner text of the element
```

The selector returns an array, generally, should be similar to BeautifulSoup, pretty intuitive.

## Getting info from elements

You can get info from attributes:

```ruby
image_source = doc.css("img").first.attribute("src").value
```