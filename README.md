---
tags: rails, javascript, ajax, jquery
languages: ruby, javascript
resources: 3
---

# Simple Todo List With Ajax

## Description

This readme will walk you through the process of adding Ajax to a very simple rails todo list app. Instead of using `remote: true` right off the bat, since we want to get a better understanding of how Ajax works, we'll be writing out our own jQuery Ajax request to add items without reloading the page. Then, we'll refactor with `remote: true`.

## The Basic Rails Todo App

Before getting to Ajax, here are the steps to create the basic rails todo app we'll be building on:

1. `rails new todo_app`
2. `rails g resource todo`
3. In the migration file (located in `db/migrate`):

  ```ruby
  class CreateTodos < ActiveRecord::Migration
    def change
      create_table :todos do |t|
        t.string :description
        t.string :priority
        t.timestamps null: false
      end
    end
  end
  ```

4. In `app/controllers/todos_controller.rb`:

  ```ruby
  class TodosController < ApplicationController
    def index
      @todos = Todo.all
    end

    def create
      Todo.create(todo_params)
      redirect_to root_path
    end

    def destroy
      todo = Todo.find(params[:id])
      todo.destroy
      redirect_to root_path
    end

    private
      def todo_params
        params.require(:todo).permit(:description, :priority)
      end
  end
  ```

5. In `config/routes.rb`:

  ```ruby
  Rails.application.routes.draw do
    root 'todos#index'
    resources :todos
  end
  ```

6. Create `views/todos/index.html.erb` with a basic form and list of todos:

  ```ruby
  <h1>My Todos</h1>

  <%= form_for Todo.new do |f| %>
    <div class="form-group">
      <%= f.text_field :description, placeholder: 
      "what needs doing?" %>
    </div>

    <div class="form-group">
      <%= f.text_field :priority, placeholder: "priority level" %>
    </div>

    <div class="form-group">
    <%= f.submit %>
    </div>
  <% end %>

  <ul>
  <% @todos.each do |todo| %>
    <li>
      <%= todo.description %><br>
      <strong>priority: </strong><%= todo.priority %><br>
      <%= link_to "done", todo_path(todo), method: 'delete' %>
    </li>
  <% end %>
  </ul>
  ```

7. Before starting up the rails server: `rake db:migrate`.

Now we can create new todos and destroy them. This is as functional as our app needs to be. Except for one thing. Since this is a single page app, it would be really nice if we could add and remove todos without reloading the page. Enter Ajax!

## Adding Ajax

### Posting a new Todo with Ajax

First we are going to "ajaxify" the process of creating todos. Since we are entering JavaScript land, we'll go into `app/assets/javascripts`. You'll see a file called `todos.coffee` or `todos.js.coffee`. We will not be using CoffeeScript, so rename it to `todos.js`. 

Here's what we are going to implement:

When the "Create Todo" button is clicked, we will stop the default form submission behavior from happening, and send an Ajax request instead. To break this down further, here are the steps we'll need to follow:

1. Create an event listener for the 'click' event.
2. Grab some information from the form to use in our Ajax request.
3. Make the Ajax request.
4. Handle the response and add the new todo to the page.

#### Create an Event Listener
In `app/assets/javascripts/todos.js`:

1. Make sure the document is ready before doing anything else.
  ```javascript
  // This is shorthand for $( document ).ready(function() { })
  $(function(){
    
  });
  ```

2. Listen for the submission of the form.
  ```javascript
  $(function(){
    $("form").submit(function(){
      // this debugger should be hit when you click the submit button!
      debugger
    });
  });
  ```

3. Prevent the default behavior (the form submitting and the page reloading).
  ```javascript
  $(function(){
    $("form").submit(function(event){
      event.preventDefault();

      // this debugger should be hit when you click the submit button!
      debugger
    });
  });
  ```

#### Grab Information From the Form
In order to make the Ajax request, we'll need to give the request the correct action and method to take us to the create action in the todos controller. If we `rake routes`, we'll see this:

`POST   /todos(.:format)          todos#create`

So we know we will be making a 'post' request to '/todos'. We could hard code this in our Ajax request, but what if our routes change later on? Our request would no longer work. In order to make our code more robust, we'll get the correct action and method directly from the form itself. Basicaly, we'll ask the form where it was going by extracting the information with jQuery.

The `debugger` we threw into our code above will be really useful in figuring out the jQuery code for pulling out the information we need. With your server running and your JavaScript console open, fill in the form and hit the submit button. When the debugger gets hit, what is `this`?

![`this` in the console](./images/js-console-1.png)

Since our debugger is inside the function that's attached to the form submission, `this` is the form itself! And the action and method we need are right there. Let's use jQuery to grab them, using the handy jQuery [.attr() method](https://api.jquery.com/attr/).

![`.attr()` in the console](./images/js-console-2.png)

```javascript
$(function(){
  $("form").submit(function(event){
    event.preventDefault();

    var action = $(this).attr('action');
    var method = $(this).attr('method');

  });
});
```

Of course, we also need the text we entered into the form for the todo description and priority level. If we open up the form element and all the divs that live inside of it in our console, we see that Rails gave our inputs some nice ids that we can use to select them with. Specifically, there's an input with an id of "todo_description" and an input with an id of "todo_priority". Lets grab these using the jQuery [.find()](https://api.jquery.com/find/) method.

![.find() in the console](./images/js-console-3.png)

We don't see the values that we typed into the form yet, but we can pull them out using the jQuery [.val()](https://api.jquery.com/val/) method.

![.val() in the console](./images/js-console-4.png)

Great! Let's store these values as variables so we can pass them in to our Ajax request.

```javascript
$(function(){
  $("form").submit(function(event){
    event.preventDefault();

    var action = $(this).attr('action');
    var method = $(this).attr('method');

    var description = $(this).find('#todo_description').val();
    var priority = $(this).find('#todo_priority').val();

  });
});
```
We are ready for the next step!

#### Make the Ajax Request
We are going to make an Ajax request that will deliver a JavaScript response to the create action in the todos controller.

The [jQuery documentation for writing Ajax requests](http://api.jquery.com/jquery.ajax/) is really nice, so here's some code I pulled from there:

```javascript
$.ajax({
  method: "POST",
  url: "some.php",
  data: { name: "John", location: "Boston" }
});
```

If we change the method, url, and data to the variables we created, we should be good to go! Our todos.js should now look like this:

```javascript
$(function(){
  $("form").submit(function(event){
    event.preventDefault();

    var action = $(this).attr('action');
    var method = $(this).attr('method');

    var description = $(this).find('#todo_description').val();
    var priority = $(this).find('#todo_priority').val();

    $.ajax({
      method: method,
      url: action,
      data: { description: description, priority: priority }
    });

  });
});

```

Since we're expecting this Ajax request to hit the create action in the todos controller, let's put a `binding.pry` in there (make sure you've added `gem 'pry'` to your Gemfile and run bundle install first):

```ruby
def create
  # if our ajax request works, we'll hit this binding at take a look at params!
  binding.pry
  Todo.create(todo_params)
  redirect_to root_path
end
```

Here's what you'll see when that binding gets hit:

![params in pry](./images/pry-1.png)

Success! The Ajax request went where we wanted it to go and it sent the params through the way we told it to. But params is not quite right. Since we're using strong params, we need a nested structure where "todo" is a top level key. By changing our Ajax request to include `data: { todo: {description: description, priority: priority} }` this problem is solved, but there's actually a jQuery method, [.serializeArray()](https://api.jquery.com/serializeArray/), that will take care of turning all our form data into a nicely structured object (nesting included!) that we can use in our Ajax call. Here's how it looks in our code:

```javascript
// .serializeArray() can be called on any form element (and here, $(this) is our form)
var data = $(this).serializeArray();

$.ajax({
  method: method,
  url: action,
  data: data
});
```

Now our params will be structured the way Rails expects them to be, and we can move on to the next step!

#### Handle the Response

We're still in the todos controller create action, and we're now able to instantiate new todos using the nicely structured params that we sent via Ajax. Currently, after creating the new Todo, we are just redirecting to the root_path. But the whole point of using Ajax is *not* to redirect. By default, a rails controller action handles responses that are sent in as html strings. We now want to tell it to respond to JavaScript:

```ruby
def create
  Todo.create(todo_params)
  
  respond_to do |format|
    # if the response fomat is html, redirect as usual
    format.html { redirect_to root_path }

    # if the response format is javascript, do something else...
    format.js { }
  end
end
```

Before going further into what Rails will do with this JavaScript response, there's one more problem with our Ajax request. We did not tell it to send back JavaScript! Let's do that now, and our Ajax call should look like this:

```javascript
$.ajax({
  method: method,
  url: action,
  data: data,

  // this line makes the response format JavaScript and not html.
  dataType: 'script'
});
```

So now that we're getting the response we want, what is `format.js { }` actually doing?

When Rails sees that line of code, it will automatically look for a file with the path `app/views/<controller name>/<action name>.js.erb`. In our example, it's looking for app/views/todos/create.js.erb.

Make that file and throw a debugger in there to confirm that it's actually where we land when we get out of the create action.

![new create.js.erb file](./images/create-js-erb-1.png)
![debugger in create.js.erb](./images/js-console-5.png)

That worked! So now that we are in create.js.erb, we have access to any instance variables we created in the controller action that led us there, and we can write JavaScript and ERB (embedded Ruby) code to do whatever we want with our view. We should probably save our newly created todo in an instance variable for just that purpose.

```ruby
def create
  @todo = Todo.create(todo_params)

  respond_to do |format|
    format.html { redirect_to root_path }
    format.js { }
  end
end
```

So what do we want to do next? In create.js.erb, we want to use JavaScript (or jQuery) to append the new todo to the `<ul>` where the todos live on the page.

We could do something like this:

```javascript
var html = "<li><%= @todo.description %><br><strong>priority: </strong> <%= @todo.priority %><br><%= escape_javascript link_to 'done', todo_path(@todo), method: 'delete' %></li>";

$('ul').append(html);
```

Now, when we enter a new todo description and priority and hit the submit button, it appears on the page and no reloading happens! That's great, but it could use some refactoring.

1. It's not DRY: we basically copied and pasted code from of our index.html.erb. If the way we want to render todos ever changes, we now have to change it in two places.
2. It's ugly. That html string is pretty hard to read, and what's that `escape_javascript` doing in there? We need that or the link_to will be evaluated as JavaScript and the whole thing breaks.

The solution to this? A partial. Extract the code for displaying a single todo out of index.html.erb and place it in  a new file at `app/views/todos/_todo.html.erb`:

![todo partial](./images/todo-partial.png)

Now you can use Rails magic to render the `@todos` collection in index.html.erb:

![index using todo partial](./images/index-with-render-collection.png)

Finally, replace the html string we had in create.js.erb with code for rendering the partial, passing in `@todo` as a local variable:

![create.js.erb using partial](./images/create-js-erb-2.png)

And we're done creating todos with Ajax! In the next section, we'll refactor our code by using `remote: true`.

### Refactoring with `remote: true`

In Rails, both `form_for` and `link_to` elements can take an argument of `remote: true`:

#### Example: `link_to` with `remote: true`
```ruby
<%= link_to 'Update Something', edit_something_path(@something), remote: true %>
```

In the case of our todo list app, we will add `remote: true` to our form for creating a new todo (the only change here is on the first line of the form):

```ruby
<%= form_for Todo.new, remote: true do |f| %>
  <div class="form-group">
    <%= f.text_field :description, placeholder: 
    "what needs doing?" %>
  </div>

  <div class="form-group">
    <%= f.text_field :priority, placeholder: "priority level" %>
  </div>

  <div class="form-group">
  <%= f.submit %>
  </div>
<% end %>
```
So, what does `remote: true` do for you? In short, it adds a `data-remote="true"` attribute to the generated html form, and submits the form via Ajax automagically. As with everything in Rails, there's metaprogramming going on under the hood. In this case, the JavaScript code that we wrote to hijack the click event and make the appropriate Ajax request is all generated for you behind the scenes. If you're feeling extra curious, check out the [Rails.js source code](https://github.com/rails/jquery-ujs/blob/148571ded762f22ccca84db38d4b4d56853ab395/src/rails.js).

Here's a small snippet from the source code linked to above. Not so surprisingly, it looks a lot like the code we wrote (especially lines 2-4!):

```javascript
if (element.is('form')) {
  method = element.attr('method');
  url = element.attr('action');
  data = element.serializeArray();
  // memoized value from clicked submit button
  var button = element.data('ujs:submit-button');
  if (button) {
    data.push(button);
    element.data('ujs:submit-button', null);
  }
}
```

Because Rails is providing all of this JavaScript for us, now that we've added `remote: true` to the form, we can get rid of everything in our todos.js file.

#### Before adding `remote: true`:
```javascript
// app/assets/javascripts/todos.js

$(function(){
  $("form").submit(function(event){
    event.preventDefault();

    var action = $(this).attr('action');
    var method = $(this).attr('method');
    var data = $(this).serializeArray();

    $.ajax({
      method: method,
      url: action,
      data: data,
      dataType: 'script'
    });

  });
});
```

#### After adding `remote: true`:
```javascript
// app/assets/javascripts/todos.js

```

Just like when we manually wrote out the Ajax call, when the form is submitted the default behavior will be prevented and a JavaScript response will be sent to the create action in the todos controller. It will then go to app/views/todos/create.js.erb, just as it did before. We will leave the rest of our code as is, and we are done with our refactoring!

### Bonus

Test out your new skills by adding Ajax functionality for deleting todo items! Try following the steps outlined in the first section of this walkthrough and writing out your Ajax request by hand, and then abstract it all away with `remote: true`. Have fun ajaxifying all the things!

![Ajaxify all the things!](./images/ajaxify-all-the-things.jpg)

## Resources

- [jQuery Documentation - jQuery.ajax()](http://api.jquery.com/jquery.ajax/)
- [Learning About Ajax](http://blog.flatironschool.com/post/56146220219/learning-about-ajax)
- [Rails Documentation - Working with JavaScript in Rails](http://guides.rubyonrails.org/working_with_javascript_in_rails.html)
