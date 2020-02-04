# Views
Each view needs to be named according to the set route/HTTP verbs & naming convention
```ruby
#Examples
index.html.erb
show.html.erb
new.html.erb
edit.html.erb
```

# Naming Static Routes
You may also customize presumably non-dynamic page file names.
```ruby
# EXAMPLES found in ~/app/views/admin/:
dashboard.html.erb
stats.html.erb
financials.html.erb
settings.html.erb
```

These custom naming conventions must be specified in `app/config/routes.rb`
```ruby
Rails.application.routes.draw do
  get 'admin/dashboard'

  get 'admin/stats'

  get 'admin/financials'

  get 'admin/settings'
end
```

These will allow you to navigate to any admin route such as: `http://your-rails-server/admin/dashboard`.
