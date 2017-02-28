# whatsa

## The basics

### `whatsa whatsa`?

Whatsa is a CLI app that allows you to search a term or phrase and receive a quick summary about that subject.

### How does it do that?

It searches your term on Wikipedia and finds the page you're most likely looking for. Whatsa gives you the first paragraph of that article, then lists the sections on the page ("History", "Early Life", "Uses", etc.). You can select one of those sections, if you'd like, and it will give you the first paragraph of that section, too. If that's not enough for you, you can ask for more, and it will give you the full section.

## Using whatsa

Enter `whatsa` in a terminal. It will ask you what you'd like to learn about, and you can enter a word or phrase. It will give you a brief summary, and a list of more specific categories about that subject, which you can select by number. If the summary it gives you isn't enough, type 'more' to read the full section.

## Anything else?

### Wishy-washy plans

May or may not use the `summarize` ruby gem when all is said and done, instead of using the first paragraph. Might be a pain to install, though, so I'm keeping it simple. 

## So what's yer structure, bucko?

Well, let's see.

- We'll have a `CLI` class that dictates the interface
- We'll have a `Searcher` class which grabs the page you're looking for (maybe just do this in the Article class?)
	- initialized with a term
	- stores the HTML document as a nokogiri object
	- yeah never mind this is just gonna be in the `Article` class
- We'll have an `Article` class which provides an API for handling an article
	- initialized with a search term
	- stores the HTML document as a nokogiri object
	- methods:
		- `#disambiguation?` - returns true if it's a disambiguation page, false if not
		- `#article?` - opposite of `#disambiguation?`
		- `#summary` - returns the first paragraph of the article
		- `#full_text` - returns the full introductory section
		- `#toc` - returns a list of section names (corresponding to the keys of the `#contents` hash)
		- `#contents` - returns a hash of section names, each pointing to an array of their paragraphs (maybe make the sections objects)
		- `#title` - returns the article title
		- `#dmb_choices` - returns a list of page titles the disambiguation page refers to, or `nil` if it's an article
		- `#full_text` - returns the full text of the article, in plain text
- We'll also have a `Section` class, I think, which handles a section of an article
	- initialized with a section name and the parent `Article` object (not necessary, just store it when the `Article` creates it)
	- stores the section name
	- stores its paragraphs in an array
	- maybe has subsections?
	- methods:
		- `#summary` - returns the first paragraph
		- `#full_text` - returns the full text of the section
		- `#title` - returns the section name

**EDIT: Everything before this point is outdated as of, like, 0.1.1**

## Bugs

- if a disambig choice name starts with a number, trying to select it by name will cause you to select that starting _number_ choice instead. I've never personally stumbled upon this case, however, so I don't care

- I think the `Whatsa::Format` module should be a class, with class methods. At the very least, the methods should be `extend`ed, not `include`d. I did it the way I did because I'm a little lazy and I didn't want to write out `self.class.word_wrap(text)` or `Whatsa::Format.word_wrap(text)` (etc.) every time I wanted to use that method.
	- I think I could cut down on this frustration if I make it a class, and then in classes I wanted to use it I could have a line that says `F = Whatsa::Format` and then just do `F.word_wrap(text)` which is much cleaner. I haven't tried that, though, but as classes are objects themselves, I don't see why that wouldn't work. But then, why have it namespaced in the Whatsa module if you never see it? It's semantic anyway. And why have them be class methods if you'll use it indistinguishably from an instantiated object? I think this is where the Object metaphor starts breaking down.

- The format module _should_ probably be in a `concerns/` directory, but since it's namespaced under `Whatsa`, I'm not sure if that's what I should do. Should I re-name it `Concerns::Format` or something similar so it's out of the `Whatsa` namespace? It does share functionality specifically with Whatsa, though. It has specific citation-removing, URL-friendlying, etc. methods that fall under Whatsa's jurisdiction, meaning that it's not really its own entity. Maybe this is another argument to make it a class with methods. I do like the fact that it's a module, though, for some reason... Am I just stubborn?