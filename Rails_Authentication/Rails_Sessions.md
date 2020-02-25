# What are sessions in Ruby on Rails?
Sessions, if you're at all familiar with Sinatra, work similarly in Rails. The idea here is that we're keeping track of information between HTTP requests. There are three main areas that you need to know/remember to work with concernings sessions. These are `controllers`, `views`, and the `routes`.

Let's take a closer look at how sessions work... We are going to use shopping carts for the example. The first thing we need to look at is the `application_controller.rb`:

```ruby
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  helper_method :cart

  def cart 
    session[:cart] ||= []
  end 
end
```

Here, we have included our `helper_method :cart` in reference to `def cart`. Within our `cart` method, we have `session[:cart] ||= []` which either keeps track of our cart session or creates an empty cart if one doesn't exist. Next, we need to consider what we are putting inside of our cart... Naturally, a `product` goes inside of our `cart`. Therefore, we need to create a `products_controller.rb`. Let's take a quick dive into what that controller will look like:

```ruby
# app/controllers/products_controller.rb

class ProductsController < ApplicationController
    def index 
    end 

    def add
        cart << params[:product]
        redirect_to '/'
    end 
end 
```

In this controller, we are defining our `/products` route via `def index`. Additionally, we are defining our `PUT` request with `def add` which is taking care of redirecting us to the main `index` page. This AFTER pushing our `params[:product]` into our `cart` array (again, defined in our `helper_method :cart` inside of our `ApplicationController`).

## Side note on views...

**NOTE: This is a random reminder that (in this case) `views/layouts` and `views/products` must exist. Below are examples of our main `application.html.erb` in `layouts` and `index.html.erb` in `products` :**

```erb
# app/views/layouts/application.html.erb

<!DOCTYPE html>
<html>
<head>
  <title>CookiesAndSessionsLab</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>

<%= yield %>

</body>
</html>
```

```erb 
# app/views/products/index.html.erb

<%= form_tag products_path do %>
    <%= text_field_tag :product %>
    <%= hidden_field_tag :authenticity_token, form_authenticity_token %>
    <%= submit_tag "add to cart" %>
<% end %>

<h2>Products</h2>
<ul>
<% cart.each do |product| %>
  <li><%= product %></li>
<% end %>
</ul>

```

# The Routing
This is the easiest part! Check out the code below found in `app/config/routes.rb` :

```ruby 
Rails.application.routes.draw do
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html

  root 'products#index'
  post '/products' => 'products#add'
end
```

Right out of the gate, we are setting our `root` directory to our `products` index page located at `app/views/products/index.html.erb`. Additionally, we've taken care of our `POST` request which takes care of executing our `add` method from our `ProductsController` which will also `redirect_to` our main `products#index` page `/products`.

# Conclusion 
Now we understand how these sessions work. At least in the context of shopping carts, we're able to submit a new `product` to our `cart` and be redirected to the main `products#index` page where we will find a list of all items in our cart. Our `session[:cart]` is easily carried from one HTTP request/page to the next without losing this information during the redirections / routing.