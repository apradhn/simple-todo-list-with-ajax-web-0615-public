---
tags: rails, javascript, ajax, jquery
languages: ruby, javascript
resources: 2
---

# Simple Todo List With Ajax

## Description

This readme will walk you through the process of adding Ajax to a very simple rails todo list app. Instead of using `remote: true`, we'll be writing out our own jQuery Ajax requests and go through the whole process step by step. By the end of this tutorial, here's the functionality we'll have:

<!-- INSERT DEMO HERE -->

## The Basic Rails Todo App

Before getting to Ajax, here are the steps that created the basic rails todo app we'll be building on:

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

Now we can create new todos and destroy them. This is as functional as our app needs to be. Except for one thing. Since this is a single page app, it would be really nice if we could add and remove todos without refreshing the page. Enter Ajax!

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
  
})
```

2. Listen for the submission of the form.
```javascript
$(function(){
  $("form").submit(function(){
    // this debugger should be hit when you click the submit button!
    debugger
  })
})
```

3. Prevent the default behavior (the form submitting and the page reloading).
```javascript
$(function(){
  $("form").submit(function(event){
    event.preventDefault();

    // this debugger should be hit when you click the submit button!
    debugger
  })
})
```

#### Grab Information From the Form
In order to make the Ajax request, we'll need to give the request the correct url and method to take us to the create action in the todos controller. If we `rake routes`, we'll see this:

`POST   /todos(.:format)          todos#create`

So we know we will be making a 'post' request to '/todos'. We could hard code this in our Ajax request, but what if our routes change later on? Our request would no longer work. In order to make our code more robust, we'll get the correct method and url directly from the form itself. Basicaly, we'll ask the form where it was going by extracting the url and method with jQuery.

The `debugger` we threw into our code above will be really useful in figuring out the jQuery code for pulling out the information we need. With your server running and your JavaScript console open, fill in the form and hit the submit button. When the debugger gets hit, what is `this`?

![`this` in the console](./images/js-console-1.png)


### Deleting a Todo with Ajax

## Resources

- [jQuery Documentation - jQuery.ajax()](http://api.jquery.com/jquery.ajax/)
- [Learning About Ajax](http://blog.flatironschool.com/post/56146220219/learning-about-ajax)
