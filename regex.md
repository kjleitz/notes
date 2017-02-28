#Regex notes

---

[toc]

---

##The basics

###Making one

You can do a regular expression `/likethis/`, between forward slashes. That is the literal form of doing it `Regexp.new('likethis')`.

###Cheat sheet

#### 'Real' characters

regex | description
--- | ---
`[abc]` | A single character of: a, b, or c
`[^abc]` | Any single character except: a, b, or c
`[a-z]` | Any single character in the range a-z
`[a-zA-Z]` | Any single character in the range a-z or A-Z
`.` | Any single character
`\s` | Any whitespace character
`\S` | Any non-whitespace character
`\d` | Any digit
`\D` | Any non-digit
`\w` | Any word character (letter, number, underscore)
`\W` | Any non-word character
`(...)` | Capture everything enclosed
`(a|b)` | a or b
`a?` | Zero or one of a
`a*` | Zero or more of a
`a+` | One or more of a
`a{3}` | Exactly 3 of a
`a{3,}` | 3 or more of a
`a{3,6}` | Between 3 and 6 of a

#### 'Anchor' characters

regex | description
--- | ---
`^` | Start of line
`$` | End of line
`\A` | Start of string
`\z` | End of string
`\b` | Any word boundary
`\B` | Any non-word boundary
`\G` | Point where last match finished
`(?=pat)` | Positive lookahead: the next characters match pat
`(?!pat)` | Negative lookahead: the next characters don't match pat
`(?<=pat)` | Positive lookbehind: the preceding characters match pat
`(?<!pat)` | Negative lookbehind: the preceding characters don't match pat



##Methods

###`#scan`

Returns an array of all matching items

###`#match`

Returns the _first_ match (as a `MatchData` object), otherwise `nil`. Can use this as a boolean to determine whether the pattern exists.

###`#grep`

Like scan, but it's an enumerable method, so you'd use it on an array of strings, for example (returns the items from an array that match the given regex).

##Tools

###Capture groups `(...)`

Use parentheses in the regexes in your scan/match/grep methods to group those portions into "capture groups" in an array. Breaks things down into discrete subgroups. Hard to explain. Check it out:

```ruby
numbers = "202-555-0192 202-555-0147 202-555-0131 202-555-0116 202-555-0192 202-555-0197"
 
number_breakdown = numbers.scan(/(\d+)-(\d+)-(\d+)/)
=> [["202", "555", "0192"], ["202", "555", "0147"], ["202", "555", "0131"], ["202", "555", "0116"], ["202", "555", "0192"], ["202", "555", "0197"]] 
 
number_breakdown[0]
=> ["202", "555", "0192"]
 
number_breakdown[0][1]
=> "555"
```

