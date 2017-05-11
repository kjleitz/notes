# AJAX notes

## The basics

AJAX: Asynchronous Javascript And XML

Using XMLHttpRequests to get data from a remote source (on the server or elsewhere) without refreshing the page/redirecting the user. Noice.

## With vanilla JavaScript

See the [XMLHttpRequest notes](xmlhttprequest.md). Pretty simple.

## With jQuery

### Making a request

This is mostly taken from [this Learn.co lesson](https://github.com/learn-co-students/js-ajax-callbacks-readme-v-000):

Say we have an `index.html` like this:

```html
<body>
  <p id="sentences">
    Loading our page...
  </p>
  <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/3.1.0/jquery.min.js"></script>
  <script src="script.js"></script>
</body>
```

...and we have another HTML file with some sentences in it:

```html
<p>Ajax is a brand of cleaning products, introduced by Colgate-Palmolive in 1947 for a powdered household and industrial cleaner.</p>
<p>It was one of the company's first major brands.</p>
```

Now, we can grab that file and insert the data into the page with jQuery's `$.get()` method (and a callback to handle the `response`):

```js
// We should wait for the page to load before running our Ajax request
$(document).ready(function(){
  // Now we start the Ajax GET request. The first parameter is the URL with the data.
  // The second parameter is a function that handles the response.
  $.get("sentence.html", function(response) {
    // Here we are getting the element on the page with the id of sentences and
    // inserting the response
    $("#sentences").html(response);
  });
});
```

The argument passed to the callback is the `responseText` of the response object, if I'm not mistaken... the data that the server sends back.

If you fire up a server with `python -m SimpleHTTPServer` and navigate to `http://localhost:8000`, you can see the AJAX in action.

You can use this technique to request data from some external API, or some other file or server or whatever, doesn't have to be on your own server. Good stuff.

Apparently you can also do the same kind of thing like this:

```js
var url = "https://api.github.com/repos/rails/rails/commits?sha=82885325e04d78fb7ec608a4670164d842d23078";
 
$.get(url)
  .done(function(data) {
    console.log("Done");
    console.log(data);
  });
```

Chain the `done()` method with a callback to fire when the response has finished coming in. Don't know what the difference is, here.

### Handling a failed request

Check it out:

```js
$.get("this_doesnt_exist.html", function(data) {
  // This will not be called because the .html file request doesn't exist
  doSomethingGood();
}).fail(function(error) {
  // This is called when an error occurs
  console.log('Something went wrong: ' + error);
});
```

Chain the `fail()` method to the `get()` request to give a callback to handle an error.