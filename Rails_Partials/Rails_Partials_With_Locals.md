# Partials with Locals

## Introduction

Partials help us break our code up into reusable chunks.  They also often have
implicit dependencies that can lead to bugs.  For example, what if a partial
assumes that a `@user` variable is present. If the point is to _reuse_ partials,
if you put it inside of an action that _didn't_ set a `@user` variable, you're
going to have a bug. Using "locals" in partials is how we can make these
implicit assumptions explicit.  In the following example, we'll unpack exactly
what locals are and how they're used.

## Lesson

Take a cool at this easily reused code:

```erb
<ul>
  <li> <%= @author.name %></li>
  <li> <%= @author.hometown %></li>
</ul>
```

You'll find that code (or very similar code) in the views:

* `app/views/authors/show.html.erb`
* `app/views/authors/index.html.erb`
* `app/views/posts/show.html.erb`.
* `app/views/posts/index.html.erb`.

Let's see how we might be vulnerable to bugs. In this `<ul>` we ***assume***
that there will be a controller-set variable, `@author`. But what if that
person-like entity makes more sense to be called `@admin` or `@guest` or
`@owner`. We want the same bit of UI, but don't want to have to re-name our
variables to make it work. We know what we want in the partial (the `<ul>`),
what we want to be _flexible_ is the "thing" that we invoke `.name` and
`hometown` on.

> **ASIDE**: This should recall the "why do methods have arguments and
> parameters" discussion from when you were learning to write methods.

Let's start with the author show page.  Watch our _process_ here as we're going
to apply it to all views that reference this `name` and `hometown` information.

Let's remove the code from our `app/views/authors/show.html.erb` page.  Now our
file should be empty:

```erb
<!-- app/views/authors/show.html.erb -->

```

We can move the removed code into a partial, `app/views/authors/_author.html.erb`, that now has the following code:

```erb
<!-- app/views/authors/_author.html.erb -->

<ul>
  <li> <%= @author.name %></li>
  <li> <%= @author.hometown %></li>
</ul>
```

To keep our code in the show page rendering out the same content, we call the
partial from the `app/views/authors/show.html.erb` file.  Doing this, the
`app/views/authors/show.html.erb` file now looks like the following:

```erb
<%= render 'author' %>
```

Great!

Now let's take a look at the `app/views/posts/show.html.erb` file.  It
currently looks like the following:

```erb
Information About the Post
<ul>
  <li> <%= @author.name %></li>
  <li> <%= @author.hometown %></li>
</ul>
<%= @post.title %>
<%= @post.content %>
```

You can see that lines 2-5 are exactly the same as the code in our
authors/author partial.  Let's remove the repetition in our codebase by using
that partial instead.  By using the partial, our code will look like the
following:

```erb
Information About the Post
<%= render 'authors/author' %>
<%= @post.title %>
<%= @post.content %>
```

> **NOTE**: Because we are calling a partial from outside the current
> `app/views/posts` folder, we must specify the folder that our author partial
> is coming from by calling `render 'authors/author'`.

## The Problem

In `app/views/authors/show.html.erb` our source of information about  `.name`
and `.hometown` is `@author`; in `app/views/posts/show.html.erb` the source of
information about `.name` and `.hometown` is `@post.author`. If we could tell
the partial "use as your source" `@author` or `@post.author`, we could share the
partial across these two different views.

The `locals` parameter to `render` provides this flexibility.

Let's see how local variables make our code more explicit.

This is what the entire show view, `app/views/posts/show.html.erb`, looks like
when `locals` are used:

```erb
Information About the Post
<%= render partial: "authors/author", locals: {post_author: @author} %>
<%= @post.title %>
<%= @post.content %>
```

Notice a few things:

1. We are no longer passing the render method a `String`; we're passing key-value pairs
2. The first key-value pair tells Rails the name of the partial to render (`"authors/author"`)
3. The second key-value pair specifies the `locals` as a `Hash`. That
   `Hash`'s keys (`post_author` here) will be created as local variables
   _within the partial_.

When we use locals, we need to make sure that the variables we refer to in our
partial have the same names as the keys in our locals hash.

In our example partial, `app/views/author/_author.html.erb`, we need to change
our code from:

```erb
<ul>
  <li> <%= @author.name %></li>
  <li> <%= @author.hometown %></li>
</ul>
```

to:

```erb
<ul>
  <li> <%= post_author.name %></li>
  <li> <%= post_author.hometown %></li>
</ul>
```

The way we use locals with a partial is similar to how we pass arguments
into a method.  In the `locals` `Hash`, the `post_author:` key is the argument
name, and the value of that argument, `@author`, is the value
stored as `post_author` and passed into the method.  We can name the keys
whatever we want.

Now notice that, if we choose to delete the line `<%= render {partial:
"authors/author", locals: {post_author: @author}} %>` from the posts/show view,
calling the partial requires us to pass in data about the author. The `@author =
@post.author` line in our `PostsController` may no longer be needed.

In fact, with locals, we can completely eliminate the `@author = @post.author`
line in the `posts#show` controller action, instead only accessing that data
where we need it, in the partial.

Let's remove that line of code in our controller and in the view pass through
the author information by changing our code to the following:

`app/controllers/posts_controller`:

```ruby
  ...
  def show
    @post = Post.find(params[:id])
  end

```

`app/views/posts/show.html.erb`:

```erb
Information About the Post
<%= render partial: "authors/author", locals: {post_author: @post.author} %>
<%= @post.title %>
<%= @post.content %>
```

This code is much better.  We are being more explicit about our dependencies,
reducing lines of code in our codebase, and reducing the scope of the author
variable.

Don't worry if you find the syntax for rendering a partial hard to remember ––
it is.  You can always reference this guide or the Rails Guides.

## Conclusion

In this lab we've learned how partials help us DRY out our views and how the
`locals` `Hash` can be used to create flexibility in our calls to the partials.
