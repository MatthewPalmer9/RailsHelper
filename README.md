# RailsHelper
A README.md to keep personal notes for myself about Rails.

# How To Start a New Rails
type 'rails new app-name'

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
Here, we are setting the name= and id= in the form to correct naming parameters.
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
