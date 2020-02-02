# RailsHelper
A README.md to keep personal notes for myself about Rails.

# How To Start a New Rails
type `rails new app-name`

# Rails form_tag And How To Use It
```ruby
<h1>Student Form</h1>
<%=form_tag students_path do%>
  <label>First Name</label><br>
  <%=text_field_tag :'student[first_name]' %><br>
  <label>Last Name</label><br>
  <%=text_field_tag :'student[last_name]' %><br>

  <%= hidden_field_tag :authenticity_token, form_authenticity_token %>
  <%=submit_tag "Submit Student"%>

  <%= params.inspect %>
<%end%>
```
Here, we are setting the `name=` and `id=` in the form to correct naming parameters.
These parameters are passed into the create method in a student_controller.rb
Looks like this:
```ruby
def create
  @student = Student.create(first_name: params[:student][:first_name], last_name: params[:student][:last_name])
  @student.save
  redirect_to student_path(@student)
end
```

# How to Set Routes
Go to YourAppName/config/routes.rb and apply this code:
```ruby
Rails.application.routes.draw do
  resources :students, only: [:index, :new, :create, :show, :edit, :destroy]
end
```

# Views
Each view needs to be named according to the set route/HTTP verbs & naming convention
```ruby
#Examples
index.html.erb
new.html.erb
show.html.erb
```

# How To Use link_to To Access /object_path/:id
```ruby
<div>
  <% @students.each do |student| %>
    <div><%= link_to student.to_s, student_path(student) %></div>
  <% end %>
</div>
```

The most important thing to understand here is that `link_to` is going to create
an `<a>` tag with a href attribute equal to the student id by using `student_path(student)`.

```ruby
# Example Output:
<div>
    <div><a href="/students/1">Scotty Ruth</a></div>
</div>
```

# How To Set a CREATE path without mapping to create.html.erb
All  methods in Rails search for a `view` template. `def create` in our students_controller.rb
will try to find `create.html.erb`. In order to avoid this and simply redirect a user, we use the `redirect_to` method in rails.
```ruby
#Example
def create
  @post = Post.new
  @post.title = params[:title]
  @post.description = params[:description]
  @post.save
  redirect_to post_path(@post)
end
```
