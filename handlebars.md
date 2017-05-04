# Handlebars notes

Great stuff!

## Interpolating

Use `{{something}}` syntax for values, use `{{#if something}} {{/if}}` syntax for block-level helpers.

## Making a template

Similar to [lodash](lodash.md).

```js
function loadIssue() {
  var issue = {
    state: "closed",
    number: 5,
    created_at: "2016-03-31 16:23:13 UTC",
    body: "Instructions say GET /team and POST /newteam. Rspec wants GET/newteam and POST/team."
  }
 
  var issueTemplate = document.getElementById("issue-template").innerHTML;
  var template = Handlebars.compile(issueTemplate);
  var result = template(issue);
  document.querySelector("main").innerHTML += result;
}
```

Pretty much the same as lodash, but with `Handlebars.compile()` instead of `_.template()`.

## Iterating over a template

If you have a collection, check it out. Here's using the template:

```js
function loadIssues() {
  var template = Handlebars.compile(document.getElementById("issue-template").innerHTML);
  var result = template(issues);
  document.getElementsByTagName("main")[0].innerHTML += result;
}
```

...here's the template itself:

```html
<main id="main">
  <a href="#" onclick="loadIssues();">Load Github Issue</a>
</main>
<script id="issue-template" type="text/x-handlebars-template">
  {{#each this}}
  <article>
    <header><h3>Issue #{{number}} ({{state}})</h3></header>
    <p>{{body}}</p>
    <footer><a href="{{url}}">created {{created_at}}</a></footer>
  </article>
  {{/each}}
</script>
```

When you do `{{#each this}}`, give a block, and do `{{/each}}` to end the block, you call the block for each member of the collection (`this` refers to the collection passed in) and the block is evaluated with the context of the _context object_ passed in each time (a.k.a. each issue in the list, if this is a list of issue objects like in the previous *Making a template* section). It's faster than doing it in a `for` loop in the JS file.

## Conditionals

You can do this:

```html
<script id="issue-template" type="text/x-handlebars-template">
  {{#each this}}
  <article>
    <header><h3>Issue #{{number}} ({{state}})</h3></header>
    <p>{{body}}</p>
    {{#if comments_count}}
    <p>{{comments_count}} Comments</p>
    {{else}}
    <p>No Comments Yet</p>
    {{/if}}
    <footer><a href="{{url}}">created {{created_at}}</a></footer>
  </article>
  {{/each}}
</script>
```

Note that `{{else}}` does not need a leading `#` or `/` because those only go at the beginning or end of the block. The `if` helper takes only one argument, which must be a value supplied by the context object. It goes by JS truthy/falsey rules.

You can also use `each` on a collection that's provided as a context value. Good stuff.

## Custom helpers

You can register a custom helper for the template to be able to use, like this:

```js
Handlebars.registerHelper('comment_body', function() {
  if(this.state === "closed") {
    return new Handlebars.SafeString(this.body)
  } else {
    return new Handlebars.SafeString("<strong>" + this.body + "</strong>")
  }
})
 
function loadIssues() {
  //...
}
```

The `this` in the helper is the same as the `this` that you access from within the template. Apparently, you can also pass a value to the function instead, if you aren't sure about using/can't use the context object provided by `this`. Also, `Handlebars.SafeString()` returns a string containing HTML so that the HTML doesn't get escaped. You might use this helper like so:

```html
<script id="issue-template" type="text/x-handlebars-template">
  {{#each this}}
  <article>
    <header><h3>Issue #{{number}} ({{state}})</h3></header>
    <p>{{comment_body}}</p>
    {{#if comments_count}}
    <p>{{comments_count}} Comments</p>
    {{else}}
    <p>No Comments Yet</p>
    {{/if}}
    <footer><a href="{{url}}">created {{created_at}}</a></footer>
  </article>
  {{/each}}
</script>
```

## Partials

You can create templates that you want to use as partials, and then register them as partials with handlebars. Then, you can use them with `{{> somePartial }}` syntax:

```html
<script id="main-template" type="text/x-handlebars-template">
  {{> namePartial }}
</script>
<script id="partial-template" type="text/x-handlebars-template">
  <div>{{name}}</div>
</script>
```

```js
Handlebars.registerPartial('namePartial', document.getElementById("partial-template").innerHTML)
function renderMain() {
  var template = Handlebars.compile(document.getElementById("main-template").innerHTML);
  var html = template({name: 'Gordon Ramsay'});
}
```

The partial receives the same context object as the main template calling it does.