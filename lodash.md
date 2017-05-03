# Lodash notes

Lodash is a sweet JS lib for templating and iterating and all sortsa useful stuff.

## Interpolating

Use `<%= someJavaScript %>`, or `<%- javaScript(withEscapedHTML) %>` for printing, and `<% someJavascript %>` for just eval. Sweet.

## Making a template

This kinda thing:

```js
// grab values from the inputs for a new comment
var commenter = document.getElementById("commenterName").value;
var comment = document.getElementById("commentText").value;

// make a template for the comment (ugly)
const commentTemplate = '<div class="comment">' +
  '<div class="comment-body">' + 
    '<%- comment %>' + 
  '</div>' + 
  '<div class="commenter">' +
    '<p>' +
      '<span class="posted-by">Posted By: </span>' +
      '<%- commenter %>' +
    '</p>' +
  '</div>' +
'</div>';

// make a template function for building a comment
const commentBuilder = _.template(commentTemplate);

// grab the comment area and append the new comment
const commentsDiv = document.getElementById('comments');
const commentHTML = commentBuilder({
  comment: comment,
  commenter: commenter
});
commentsDiv.innerHTML += commentHTML;
```

You can also do this sort of pattern (note the `text/x-lodash-template` type):

```html
  <script id="comment-template" type="text/x-lodash-template">
    <div class="comment">
      <div class="comment-body"><%= comment %></div>
      <div class="commenter">
        <p>
          <span class="posted-by">Posted By: </span>
          <%= commenter %>
        </p>
      </div>
    </div>
  </script>
```

...and then grab it with:

```js
...
const commentTemplate = document.getElementById('comment-template');
...
```

