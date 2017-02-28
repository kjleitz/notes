# Testing notes

## Unit tests

Unit tests test the models in your program, how they interact with the database, etc.

## Controller tests

Controller tests check to make sure that appropriate data is being correctly delivered to the user, giving the expected response to particular HTTP requests, etc. They don't test forms or HTML, but how the controller is behaving.

## Integration tests

Integration tests check the "end-to-end" process of how a user will interact with an application. They flex your entire application (models, views, and controllers) and do not isolate behaviors or pieces of the code. They are used to spec user interactions with your HTML and forms. You can use a testing library called Capybara (see [capybara.md](capybara.md)) for this.