# XMLHttpRequest

## Basic use

Say we want to grab the list of a user's repositories on github using their API. Let's tie a function to a link so we can click it and get the repos logged to the console:

```html
<a href="#" onclick="getRepositories();">Get repositories</a>
```

```js
function showRepositories(event, data) {
  console.log(this.responseText);
}

function getRepositories() {
  const req = new XMLHttpRequest();
  req.addEventListener('load', showRepositories);
  req.open('GET', 'https://api.github.com/users/kjleitz/repos');
  req.send();
}
```

In `getRepositories()`, we:

- initialize an `XMLHttpRequest` object
- add a `'load'` event listener to the request object, which will fire a callback function when a response is received
- use the `open()` method on the request object to set what kind of request it will make (the HTTP method, 'GET', and the URI for the request to be sent to)
- use the `send()` method to actually send the request (after setting it all up)

Once a response is received, `showRepositories()` logs the text of the response (`this` in this context refers to `req`, our request object). All this happens without the page ever reloading!

## Parsing JSON

This is a bad way of doing this, and JSON parsing isn't exactly the topic of these notes, but:

```js
function showRepositories(event, data) {
  var repos = JSON.parse(this.responseText)
  console.log(repos)
  const repoList = `<ul>${repos.map(r => '<li>' + r.name + '</li>').join('')}</ul>`
  document.getElementById("repositories").innerHTML = repoList
}
```