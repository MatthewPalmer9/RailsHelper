# Displaying Associations in Views

A great way to display your associations is getting a visual of how they work.

```ruby
<!-- app/views/posts/show.html.erb -->

<h1><%= @post.title %></h1>

<h3>Category: <%= link_to @post.category.name, category_path(@post.category) if @post.category %></h3>

<p><%= @post.description %></p>
```

The output of this `.erb` is:


<h1>10 Ways You Are Already Awesome</h1>
Category: Motivation
