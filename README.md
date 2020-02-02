# RailsHelper
A README.md to keep personal notes for myself about Rails.

# How To Start a New Rails
type `rails new app-name`

# Rails form_tag And How To Use It
```
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
```
def create
  @student = Student.create(first_name: params[:student][:first_name], last_name: params[:student][:last_name])
  @student.save
  redirect_to student_path(@student)
end
```

# How to Set Routes
Go to YourAppName/config/routes.rb and apply this code:
```
Rails.application.routes.draw do
  resources :students, only: [:index, :new, :create, :show, :edit, :destroy]
end
```

# Views
Each view needs to be named according to the set route/HTTP verbs & naming convention
```
#Examples
index.html.erb
new.html.erb
show.html.erb
```

# How To Use link_to To Access /object_path/:id
```
<div>
  <% @students.each do |student| %>
    <div><%= link_to student.to_s, student_path(student) %></div>
  <% end %>
</div>
```

The most important thing to understand here is that `link_to` is going to create
an `<a>` tag with a href attribute equal to the student id by using `student_path(student)`.

```
# Example Output:
<div>
    <div><a href="/students/1">Scotty Ruth</a></div>
</div>
```
