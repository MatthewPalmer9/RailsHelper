# Using `has_secure_password` in Rails

Rails provides a method called `has_secure_password` that you can use on your ActiveRecord models to handle all this. It looks like this:

```ruby
class User < ActiveRecord::Base
  has_secure_password
end
```

You'll need to add `gem 'bcrypt'` to your Gemfile if it isn't already.

[`has_secure_password`][has_secure_password] adds two fields to your model: `password` and `password_confirmation`. These fields don't correspond to database columns! Instead, the method expects there to be a `password_digest` column defined in your migrations.

`has_secure_password` also adds some `before_save` hooks to your model. These compare `password` and `password_confirmation`. If they match (or if `password_confirmation` is `nil`), then it updates the `password_digest` column pretty much exactly like our example code before did.

These fields are designed to make it easy to include a password confirmation box when creating or updating a user. All together, our very secure app might look like this:

```erb
<%# app/views/user/new.html.erb %>
<%= form_for :user, url: '/users' do |f| %>
  Username: <%= f.text_field :username %>
  Password: <%= f.password_field :password %>
  Password Confirmation: <%= f.password_field :password_confirmation %>
  <%= f.submit "Submit" %>
<% end %>
```
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def create
    User.create(user_params)
  end

  private

  def user_params
    params.require(:user).permit(:username, :password, :password_confirmation)
  end
end
```
```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def create
    @user = User.find_by(username: params[:username])
    return head(:forbidden) unless @user.authenticate(params[:password])
    session[:user_id] = @user.id
  end
end
```
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  has_secure_password
end
```
