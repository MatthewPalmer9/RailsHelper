# Routing And Nested Resources

## Objectives

1. Understand the value of nested routes
2. Create nested routes
3. Understand how nested resource URL helpers are named

## Lesson

We're going to keep working on our blog application, augmenting it to filter posts by author in a more user-friendly and RESTful way.

### URL As Data

You've encountered REST already, but, just to review, it stands for REpresentational State Transfer and encapsulates a way of structuring a URL so that access to specific resources is predictable and standardized.

In practice, that means that, if we type `rails s` and run our blog app, browsing to `/posts` will show us the index of all `Post` objects. And if we want to view a specific `Author`, we can guess the URL for that (as long as we know the author's `id`) by going to `/authors/:id`.

**Top-tip:** the `/:id` notation above represents a *dynamic* route segment, which we've touched on before and will be seeing more of in this reading.

Why do we care?

When we added the filter button to our blog, we could filter by a certain author to see that author's posts, but the URL looked something like this:

`http://localhost:3000/posts?utf8=%E2%9C%93&author=1&date=&commit=Filter`

That's the opposite of REST. That makes me *stressed*.

![Not Sorry](http://i.giphy.com/J2LrJRhCjIKTC.gif)

That URL tells us nothing, really, about the resources we're accessing. Not to mention that it's uglier than a Geocities site, and we might want to present our readers with a link they could understand and share with friends.

Then there's the author, who might want a more presentable link to share that lets people know, "Hey, these posts belong to me!"

### Dynamic Route Segments

What we'd love to end up with here is something like `/authors/1/posts` for all of an author's posts and `/authors/1/posts/5` to see an individual post by that author.

We know we can build out a route with dynamic segments, so our first instinct might be to just define these in `routes.rb` like this:

```ruby
# config/routes.rb

  get 'authors/:id/posts'
  get 'authors/:id/posts/:post_id'
```

After adding those routes, let's load up our blog with `rails s` (don't forget to `rake db:seed`) and check it out by browsing to `/authors/1/posts`.

Oops. Error. Gotta tell those routes explicitly which controller actions will handle them. Okay, let's make it look more like this:

```ruby
# config/routes.rb

  get 'authors/:id/posts', to: 'authors#posts_index'
  get 'authors/:id/posts/:post_id', to: 'authors#post'
```

And to handle our new filtering routes, we'll need to make some changes in our `authors_controller` to actually do the work.

```ruby
# app/controllers/authors_controller.rb
  def posts_index
    @author = Author.find(params[:id])
    @posts = @author.posts
    render template: 'posts/index'
  end

  def post
    @author = Author.find(params[:id])

    # Note that because ids are unique by table we can go directly to
    # Post.find — no need for @author.posts.find...
    @post = Post.find(params[:post_id])
    render template: 'posts/show'
  end
```

**Advanced:** While a controller action would normally implicitly render a template with the same name as the method, in this case we want to leverage the templates we're already using for posts, so we call `render` explicitly with a template path. Because we're telling `render` that we're using a `template`, we don't need to include the `.html.erb` extensions. Rails figures that out for us.

If we go back to our blog and try to browse to `/authors/1/posts`, we should see the posts for that author. And then if we try `/authors/1/posts/1`, we should see that post.

**Note:** If your IDs are different and you are having trouble with the URLs, try running `rake db:reset` to reset your IDs to the defaults in the seed file.

We did it! We have much nicer URLs now. Are we done? Of course not.

If we look at our `routes.rb`, we can already see it getting messy. Instead of something nice like `resources :authors`, now we're specifying controller actions and HTTP verbs just to do a simple filter of an author's posts.

Beyond that, our DRY (Don't Repeat Yourself) and Separation of Concerns klaxons should be wailing because the code to find all posts and to find individual posts by their ID is essentially repeated in the `authors_controller`. These aren't really the concerns of the `authors_controller`, and we can tell that because we're directly rendering `Post`-related templates.

Seems like Rails would have a way to bail us out of this mess.

### Nested Resource Routes

Turns out, Rails *does* give us a way to make this a lot nicer.

If we look again at our models, we see that an author `has_many :posts` and a post `belongs_to :author`. Since a post can logically be considered a *child* object to an author, it can also be considered a *nested resource* of an author for routing purposes.

Nested resources give us a way to document that parent/child relationship in our routes and, ultimately, our URLs.

Let's get back into `routes.rb`, delete the two routes we just added, and recreate them as nested resources. We should end up with something like this:

```ruby
# config/routes.rb

Rails.application.routes.draw do

  resources :authors, only: [:show] do
    # nested resource for posts
    resources :posts, only: [:show, :index]
  end

  resources :posts, only: [:index, :show, :new, :create, :edit, :update]

  root 'posts#index'
end
```

Now we have the resourced `:authors` route, but by adding the `do...end` we can pass it a block of its nested routes.

We can still do things to the nested resources that we do to a non-nested resource, like limit them to only certain actions. In this case, we only want to nest `:show` and `:index` under `:authors`.

Under that, we still have our regular resourced `:posts` routes because we still want to let people see all posts, create and edit posts, and so on outside of the context of an author.

Now we need to update our `posts_controller` to handle the nested resource we just set up. Notice how now we are dealing with the `posts_controller` rather than the `authors_controller`. Ultimately, the resource we're requesting is related to posts, so Separation of Concerns tells us to put that code in the `posts_controller`. And, since we already have actions to handle `:show` and `:index`, we won't be repeating ourselves like we did in the `authors_controller`.

Let's update `index` and `show` to account for the new routes:

```ruby
# app/controllers/posts_controller.rb

  def index
    if params[:author_id]
      @posts = Author.find(params[:author_id]).posts
    else
      @posts = Post.all
    end
  end

  def show
    @post = Post.find(params[:id])
  end
```

We added a conditional to the `posts#index` action to account for whether the user is trying to access the index of *all* posts (`Post.all`) or just the index of all posts *by a certain author* (`Author.find(params[:author_id]).posts`). The conditional hinges on whether there's an `:author_id` key in the `params` hash — in other words, whether the user navigated to `/authors/:id/posts` or simply `/posts`. We didn't have to create any new methods or make explicit calls to render new templates. We just added a simple check for `params[:author_id]`, and we're good to go.

Where is `params[:author_id]` coming from? Rails provides it for us through the nested route, so we don't have to worry about a collision with the `:id` parameter that `posts#show` is looking for. Rails takes the parent resource's name and appends `_id` to it for a nice, predictable way to find the parent resource's ID.

But, wait– we didn't make a single change to the `posts#show` action. What about the new `/authors/:id/posts/:id` route that we added? Remember, the point of nesting our resources is to DRY up our code. We had to create a conditional for the `posts#index` action because it renders *different* sets of posts depending on the path, `/authors/:id/posts` or `/posts`. Conversely, the `posts#show` route is going to render the *same* information — data concerning a single post — regardless of whether it is accessed via `/authors/:id/posts/:id` or `/posts/:id`.

For good measure, let's go into our `authors_controller.rb` and delete the two actions (`post` and `posts_index`) that we added above so that it looks like this:

```ruby
# app/controllers/authors_controller.rb

class AuthorsController < ApplicationController

  def show
    @author = Author.find(params[:id])
  end

end
```

**Top-tip:** Keep your application clean and easy to maintain by always removing unused code.

### Nested Route URL Helpers

We've got our routes working and the `posts_controller` is handling its business, but how can we present this on the page so that someone knows how to find a link to an author's posts?

Just like any other resourced route, Rails provides named helpers for our nested routes as well. And, just like most other things Rails provides, there's a predictable way to figure out what they are.

If we want to get to the `/authors` page, we know the URL helpers are `authors_path` and `authors_url`. And if we want to get to a single author (`/authors/:id`), we can use `author_path(id)`. Similarly, we have `posts_path` for `/posts` and `post_path(id)` for `/posts/:id`.

So what if we want to get to all posts nested under an author?

We know the URL is `/authors/:author_id/posts`, so we can combine the two conventions and use `author_posts_path(author_id)`. Remember it's the singular `author` because we are getting one by `id`.

It stands to reason that a single post for an author would combine the conventions for the single author path and single post path, leaving us with `author_post_path(author_id, post_id)`.

Once you become accustomed to breaking it down in that way, it's pretty straightforward to know what our URL helpers will be for a nested route. However, if you're not sure, or if you just want to double-check, you can use `rake routes` on the command line to get a printout of all your named routes, like this:

```bash
      Prefix Verb  URI Pattern                             Controller#Action
  test_index GET   /test/index(.:format)                   test#index
author_posts GET   /authors/:author_id/posts(.:format)     posts#index
 author_post GET   /authors/:author_id/posts/:id(.:format) posts#show
      author GET   /authors/:id(.:format)                  authors#show
       posts GET   /posts(.:format)                        posts#index
             POST  /posts(.:format)                        posts#create
    new_post GET   /posts/new(.:format)                    posts#new
   edit_post GET   /posts/:id/edit(.:format)               posts#edit
        post GET   /posts/:id(.:format)                    posts#show
             PATCH /posts/:id(.:format)                    posts#update
             PUT   /posts/:id(.:format)                    posts#update
        root GET   /                                       posts#index
```

You can also view this in your browser anytime you type in an incorrect route or if you visit `rails/info/routes.`

If you add `_path` or `_url` to any of the names under "Prefix", you'll have the helper for that route.

**Advanced:** Using `rake routes` can be a lot easier than browsing the `routes.rb` file once a project gets to a certain size, but the output might be overwhelming. Remember that you can `grep` the output of any command to search for what you want. So in the example above, if you just wanted to search for routes related to authors, you could type `rake routes | grep authors` to get a filtered list.

Let's make it easy for our readers to look at the posts for each of our authors using these helpers.

In `posts/index.html.erb`, we already show the author's name, so let's add a link to the list of the author's posts:

```erb
<!-- app/views/posts/index.html.erb -->

  ...

  <h2><%= post.title %></h2>
  
  <!-- change the name to a link -->
  <h3>by: <%= link_to post.author.name, author_posts_path(post.author) %></h3>
  <p><%= post.description %></p>

  ...
```
Let's reload `/posts` and click on an author name. We should be taken to `/authors/id/posts`.

Great! Now our URLs properly reflect the relationship of our resources and read almost like an English sentence: `authors/1/posts` = "author number one's posts."

Sometimes people get confused by path helpers and what they take as arguments. For example, why does `posts_path` not take an argument, but `post_path(@post)` does? `posts_path` refers to all of the posts, so we're not talking about a specific post (no specific ID). When we're looking at the `post_path`, we're referring to a specific post, and the Rails helper needs to know *which* post so that it can generate the proper url, e.g., `posts/1` as opposed to `posts/2`.

### Caveat on Nesting Resources More Than One Level Deep

You can nest resources more than one level deep, but that is generally a bad idea.

Imagine if we also had comments in this blog. This would be a perfectly fine use of nesting:

```ruby
resources :posts do
  resources :comments
end
```

We could then link to a post's comments with `post_comments_path` or `/posts/1/comments`. That makes a lot of sense.

But if we then tried to add to our already nested `posts` resource...

```ruby
resources :authors do
  resources :posts do
    resources :comments
  end
end
```

Now we're getting into messy territory. Our `comments_path` helper is now `author_post_comments_path`, our URL is `/authors/1/posts/1/comments`, and we have to handle for that filtering in our controller.

But if we lean on our old friend Separation of Concerns, we can conclude that a post's comments are not the concern of an author and therefore don't belong nested two levels deep under the `:authors` resource.

In addition, the reason to put the ID of the resource in the URL is so that we have access to it in the controller. If we know we have the post with an ID of `1`, we can use our Active Record relationships to call:

```ruby
  @post = Post.find(params[:id])
  @post.author # This will tell us who the author of the post was! We don't need this information in the URL
```

## Summary

Nesting resources is a powerful tool that helps you keep your routes neat and tidy and is better than dynamic route segments for representing parent/child relationships in your system.

However, as a general rule, you should only nest resources one level deep and ensure that you are considering Separation of Concerns in your routing.
