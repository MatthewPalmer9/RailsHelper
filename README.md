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

# How to Set Routes Using CRUD
Go to YourAppName/config/routes.rb and apply this code:
```ruby
Rails.application.routes.draw do
  resources :students, only: [:index, :show, :new, :create, :edit, :update]
  # Be sure not to add route resources like :edit and :update here to your resources if you are defining requests like below...
  # vvv
  get 'students/:id/edit', to: 'students#edit', as: :edit_students
  patch 'students/:id', to: 'students#update'
end
```
At this point, we want our `controller action` to reflect on these routes with corresponding methods...

```ruby
def edit
  @student = Student.find(params[:id])
end
```

Now that the `edit` view template will have access to the `Student` object (stored in @student), we need to refactor the form so that it auto-fills the form fields with the corresponding data from @student. We'll also use a different form helper, `form_for`, which will automatically set up the url where the form will be sent. These changes can be seen below:

```ruby
<% # app/views/students/edit.html.erb %>

<%= form_for(@student) do |f| %>
  <%= f.label 'Student First Name:' %><br>
  <%= f.text_field :first_name %><br>

  <%= f.label 'Student Last Name' %><br>
  <%= f.text_area :last_name %><br>

  <%= f.submit "Submit Student" %>
<% end %>
```
In this case, `form_for` takes care of some work for us. Using the object @student we've provided, `form_for` determines that @student is not a new instance of the `Student` class. Because of this, `form_for` knows to automatically send to the `update` path.

The `update` path will only work if the corresponding controller action reflects an update, as seen below:

```ruby
def update
  @student = Student.find(params[:id])
  # @article.update(first_name: params[:student][:first_name], last_name: params[:student][:last_name])
  @student.update(params.require(:post).permit(:title, :description))
  redirect_to student_path(@article)
end
```
So then... with `@student.update`, this is a **tough** one to retain, but the purpose of needing to `require` the `post` model is because of this: The 'params' traditionally look like this:
```ruby
{
  "first_name": "Scotty",
  "last_name": "Ruth"
}
```
But with `form_for`, our `params` now looks like this:
```ruby
{
  "student": {
            "first_name": "Scotty",
            "last_name": "Ruth"
          }
}
```
Notice how the `:first_name` and `:last_name` attributes are now nested within the `student` hash? That's why we needed to add the require method. But Rails wants us to be conscious of which attributes we allow to be updated in our database, so we must also `permit` the `:first_name` and `:last_name` in the nested hash. Using strong parameters like this will allow ActiveRecord to use mass assignment without trouble.

# Views
Each view needs to be named according to the set route/HTTP verbs & naming convention
```ruby
#Examples
index.html.erb
show.html.erb
new.html.erb
edit.html.erb
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
  @student = Student.new
  @student.first_name = params[:first_name]
  @student.last_name = params[:last_name]
  @student.save
  redirect_to student_path(@student)
end
```
In this refactored `create` action, we're following the convention of redirecting to the new resource's `show` page. It stands to reason that a user who submits a new post would then like to view the successfully-created post. *NOTE:* With this method, your `text_field_tag` will need to contain `:first_name` and `:last_name` *INSTEAD OF* `:'student[first_name]'` and `:'student[last_name]'` since the syntax is different.
