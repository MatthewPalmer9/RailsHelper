# How Would a Session Authenticate a User?

Put simply, we would be using a `SessionsController` to handle this concern. In this case, we are handling the creation of a `User`. It all looks like this:

```ruby
class SessionsController < ApplicationController
  def new
  end

  def create
    @user = User.find_by(:name => params[:user][:name])
    if @user && @user.authenticate(params[:user][:password])
      session[:user_id] = @user.id
      redirect_to '/'
    else
      redirect_to :login
    end
  end

  def destroy
  end
end
```

```ruby
class UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    if @user.valid?
      @user.save
      session[:user_id] = @user.id
      redirect_to '/'
    else
      redirect_to :signup
    end
  end

  private
  def user_params
    params.require(:user).permit(:name, :password, :password_confirmation)
  end
end
```

## What About Forms?

The form for a new **session**

```erb
<h1>Login</h1>
<%= form_for :sessions, url: '/login' do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %><br />
  <%= f.label :password %>
  <%= f.password_field :password %><br />
  <%= f.submit %>
<% end %>
```

The form for a new **user**

```erb
<h1>Sign-Up</h1>
<%= form_for :user, url: '/signup' do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %><br />
  <%= f.label :password %>
  <%= f.password_field :password %><br />
  <%= f.label :password_confirmation %>
  <%= f.password_field :password_confirmation %><br />
  <%= f.submit %>
<% end %>
```

## What of the Routes?

```ruby
Rails.application.routes.draw do
  
  root 'welcome#welcome'
  get '/signup' => 'users#new'
  post '/signup' => 'users#create'
  get '/login' => 'sessions#new'
  post '/login' => 'sessions#create'
  post '/logout' => 'sessions#destroy'

end
```

## What of the Database/Migration?

```ruby
class CreateUsers < ActiveRecord::Migration
    def change
        create_table :users do |t|
            t.string :name
            t.string :password_digest

            t.timestamps null: false 
        end 
    end 
end 
```