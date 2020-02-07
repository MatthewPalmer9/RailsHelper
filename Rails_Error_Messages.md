# Displaying All Errors with `errors.full_messages`

When a model fails validation, its `errors` attribute is filled with information about what went wrong. Rails creates an `ActiveModel::Errors` object to carry this information.

The simplest way to show errors is to just spit them all out at the top of the form by iterating over `@person.errors.full_messages.` But first, we'll have to check whether there are errors to display with `@person.errors.any?`.
```ruby
<% if @student.errors.any? %>
 <div id="error_explanation">
   <h2>
     <%= pluralize(@student.errors.count, "error") %>
     prohibited this song from being saved:
   </h2>

   <ul>
     <% @student.errors.full_messages.each do |msg| %>
     <li><%= msg %></li>
     <% end %>
   </ul>
 </div>
<% end %>

```

The code above is best placed in to `_errors.html.erb` along with your actual `form_for` form placed in `_forms.html.erb` with the rest of your `view` files. When successfully placed, we can now call to them in our view pages as seen in the following:

```ruby
# app/views/students/edit.html.erb
<%= render "songs/errors" %>
<h1>Edit Song</h1>
<%= render "songs/form" %>

# OR

# app/views/students/new.html.erb

<%= render "songs/errors" %>
<h1>Create A Song</h1>
<%= render "songs/form" %>
```

The `<%= render "file/erb-file" %>` snippet renders the content from inside `_errors` and `_form` and places that content in your `.erb` view file. This caters to what is know as DRY code. (Don't Repeat Yourself).
