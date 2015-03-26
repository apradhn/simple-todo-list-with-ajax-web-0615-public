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
3. In the migration file:

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

4. In the todos_controller:

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

5. In the config/routes.rb:

```ruby
Rails.application.routes.draw do
  root 'todos#index'
  resources :todos
end
```

6. Create views/todos/index.html.erb:
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




## Resources

- [jQuery Documentation - jQuery.ajax()](http://api.jquery.com/jquery.ajax/)
- [Learning About Ajax](http://blog.flatironschool.com/post/56146220219/learning-about-ajax)
