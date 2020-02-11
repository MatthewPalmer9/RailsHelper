# Rails Layouts and Templates

This lesson will show you how to use layouts to achieve a common look and feel between different views in your app.

## Objectives

After this lesson, you'll be able to...

1. Identify the default application layout.
2. Yield to view templates from a layout.
3. Specify a custom layout in ActionController on a controller level using the `layout` macro and on the action level using the `render :layout => "custom"` option.
4. Use :layout => false to shut off the layout and only render the view

## Life Without Layouts

Imagine that you're tasked to build an online store app with Rails.

You would probably have a few different views in this app, for example:

1. A list of products
2. A detail view that shows more info for a selected product
3. A shopping cart

Across all these views you would want a consistent look. This consistent look perhaps contains something like a logo, navigation links, a search bar, and a footer at the bottom that contains some info about the shop.

You start building the list of products, and you end up with an action template that looks something like this. It has a nav component, a list of products, and a footer. It would be the worst online shop ever, but let's keep it simple for now.

```erb
<!-- app/views/products/index.html.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Flatiron Store</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>  
</head>
<body>
    <div class="navigation">
      <ul>
        <a href="/products">Products</a>
        <a href="/cart">Cart</a>
      </ul>
    </div>

    <h1>The Product List</h1>

    <ul>
      <% @products.each do |job| %>
        <li><%= link_to 'Show', job %></li>
      <% end %>
    </ul>  

    <div class="footer">
      <p>This shop promises the lowest prices in widgets!</p>
    </div>    
</body>
</html>
```

Great, so you have a list of products now! Any shop should have that, shouldn't it? Next, you decide to build the view that will show the product details when customers click on a link for that product.

You finish sorting out the view's structure with a nav and footer. You open up `app/views/products/index.html.erb`, select all of the code, and copy it onto your clipboard. Now you are ready to paste it into your new template and customize it to show product info.

```erb
<!-- app/views/products/show.html.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Flatiron Store</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>  
</head>
<body>
    <div class="navigation">
      <ul>
        <a href="/products">Products</a>
        <a href="/cart">Cart</a>
      </ul>
    </div>

    <h1><%= @product.name%></h1>

    <p>
      <strong>Category:</strong>
      <%= @product.category %>
    </p>

    <p>
      <strong>Price:</strong>
      <%= @product.price %>
    </p>

    <div class="footer">
      <p>This shop promises the lowest prices in widgets!</p>
    </div>    
</body>
</html>
```

You call over your colleagues, all excited that you've managed to get it working so quickly. The designer takes one look and says: "You know what, the navigation is all wrong. You need to make it use the `orange` class, and also make it pop by using the `pop` class."

Oh no! You have to go and change it in two files! Imagine having tens or even hundreds of templates like this. Making the same change over and over again would be very time consuming and prone to errors. In other words, it'll be a total drag. Because we've violated the DRY principle, we have to make changes in more than one place. Layouts apply the same principle as methods to generating HTML code.

## Layouts to the Rescue

Luckily, you don't need to copy content from one template file to the next because layouts in Rails are enabled by default. When you generate a new Rails app, it generates a layout for you.

To find the generated layout, go and have a look in your Rails app at the following path. When you render a template for an action without specifying a different layout to use, Rails will use the layout found at this location: `app/views/layouts/application.html.erb`.

When you first generate a Rails app, depending on your version of Rails, the automatically generated layout file will look similar to this. The `application.html.erb` file is a very good place to start adding common components like the navigation, search, and footer from the example above.

```erb
<!-- app/views/layouts/application.html.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Flatiron Store</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>

<%= yield %>

</body>
</html>
```

## Yield

Let's say you code a new layout from scratch, ending up with something like this:

```erb
<!-- app/views/layouts/application.html.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Flatiron Store</title>
</head>
<body>
    <h1>Welcome To The Flatiron Store!</h1>
</body>
</html>
```

**Note:** usually you should include links to assets like style sheets and JavaScript files in your layouts, which was omitted in the code above to keep it simple.

Other than the missing links to common assets, this layout is missing something **terribly important**. To see what it is, have a look at the following example that uses this layout.

This is an example of an action (`static#about`) where we expect the corresponding template (`app/views/static/about.html.erb`) to render within the `application.html.erb` layout defined above.

```ruby
# app/controllers/static_controller.rb

class StaticController < ApplicationController
  def about
  end
end
```

There should also be an associated route in the `config/routes.rb` file to route a request to `/about` to the `static#about` action in the `StaticController`, above.

```ruby
# config/routes.rb

Rails.application.routes.draw do
  get 'about', to: 'static#about'
end
```

And this is the template for the `static#about` action with a simple message, which we would want to display nested inside the `application.html.erb` layout:

```erb
<!-- app/views/static/about.html.erb -->

<p>Hello!</p>
```

When you load up the `static#about` route in your browser, you will be greeted with a very bold message –– **Welcome To The Flatiron Store!** –– but you won't see the "Hello!" from the about action's template, which we just coded above. What gives?

This is happening because the layout file at `app/views/layouts/application.html.erb` does not have a `yield` statement in it.
`yield` is what Rails uses to decide where in the layout to render the content for the action. If you don't put a `yield` in your layout, the layout itself will render just fine, but any additional content coded into the action templates will not be correctly placed within the layout.

To fix this issue, add a `yield` to the layout file at `app/views/layouts/application.html.erb`:

```erb
<!-- app/views/layouts/application.html.erb -->

<!DOCTYPE html>
<html>
<head>
  <title>Flatiron Store</title>
</head>
<body>
    <h1>Welcome To The Flatiron Store!</h1>
    <%= yield %>    
</body>
</html>
```

Now when you hit up the `static#about` route in your browser, you will see **Welcome To The Flatiron Store!** followed by **Hello!** This means that, when the layout rendered, it pulled the action's specific template into the correct place –– right where we added the `yield` statement!

Those of you who remember Ruby blocks might be thinking that this looks very similar to how Ruby yields to blocks in a method call.  The same principle is at work here: we're yielding to a chunk of code and don't know what it will do in advance. We always want to run the layout part of the method, and we let any other chunk of code that produces HTML (a template in this case) run at the `yield` point in the method call.

## How Layouts and Templates are Stitched Together

At its simplest level, this is what happens when a request is made to your Rails application:

1. Rails finds the template for the corresponding action based either on convention or any other options passed to the `render` method in your controller action.
2. Similarly, it then finds the correct layout to use, either through naming/directory conventions or from specific options that you provided.
3. Rails uses the action template to generate the content specific to the action. (Note that the template might be composed of partial views, which you'll learn about a bit later.)
4. It then looks for the layout's `yield` statement and inserts the action's template there.

So this means that, for every request handled by Rails, at most one layout and action template will be used. The action template can call out to other templates, called partials, to render itself. Partials are covered in upcoming lessons, so don't worry too much about the concept for now.

## How Rails Decides Which Layout to Use

Think about the example from earlier in this lesson, the Flatiron store app. As mentioned before, it would make sense to have the same layout for the product list, product detail pages, and cart because you would want some common elements in the same place within each view.

But when you add administrative functionality to the online store –– say, in order to allow someone to add new products to the site, update prices, and perhaps draw reports –– you'll probably want to use a different layout, which is quite easy to do with Rails.

### Deciding on a Layout Through Convention

Rails uses a simple convention to find the correct layout for your request. If you have a controller called `ProductsController`, Rails will see whether there is a layout for that controller at `layouts/products.html.erb`. Similarly, if you have a controller called `AdminController`, it'll look for a layout at `layouts/admin.html.erb`. If it can't find a layout specific to your controller, it'll use the default layout at `app/views/layouts/application.html.erb`.

With the exception of the admin section of a site, most applications use the default layout for everything, so there's no need to have a layout for each controller. You want to have a consistent look and feel throughout your site, using a different layout only if the situation really warrants it.

### Overriding Conventions

If you need to override the conventions explained above, you can easily do so. For example, if you have a controller called `ShoppingCartController` and want to use the layout at `layouts/products.html.erb`, you have two options:

1. If you want to use the products layout for every action, simply specify that you want to use the products layout by invoking the `layout` method in your controller, passing it a string that it can use to find the desired layout:

  ```ruby
  class ShoppingCartController < ApplicationController
    layout "products"
  end
  ```

2. If you want to use the products layout only for a particular action, simply use the `render` method in the controller action, specifying the layout you want it to use like this:

  ```ruby
  class ShoppingCartController < ApplicationController
    def list
      render :layout => "products"
    end

    # the rest of the actions will use standard conventions
  end
  ```

If you want to render your action template without a layout, you can do the following:

```ruby
class ShoppingCartController < ApplicationController
  def list
    render :layout => false
  end

  # the rest of the actions will use standard conventions
end
```

**Note:** It's pretty unusual to not render the layout in a standard action.  However, once you start using AJAX (JavaScript), it's quite common. Keep this in the back of your mind when you get to JavaScript.

## Recap

Use layouts to provide a common look and feel between different views of your app. As much as possible, use and adapt only the default layout until you have a good reason to use a different layout for a different section or action.
