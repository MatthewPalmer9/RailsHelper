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

Here is another example for good measure:
```ruby
<%= form_tag("/cats") do %>
  <%= label_tag('cat[name]', "Name") %>
  <%= text_field_tag('cat[name]') %>

  <%= label_tag('cat[color]', "Color") %>
  <%= text_field_tag('cat[color]') %>

  <%= submit_tag "Create Cat" %>
<% end %>
```

This version of the form outputs the following HTML:
```ruby
<form accept-charset="UTF-8" action="/cats" method="POST">
  <label for="cat_name">Name</label>
  <input id="cat_name" name="cat[name]" type="text">
  <label for="cat_color">Color</label>
  <input id="cat_color" name="cat[color]" type="text">
  <input name="commit" type="submit" value="Create Cat">
</form>
```

# How to Set Routes Using CRUD & Forms
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
**`form_for` is used when directly connected with an ActiveRecord model. In this case, it is student.rb.**
The `update` path will only work if the corresponding controller action reflects an update, as seen below:

```ruby
def update
  @student = Student.find(params[:id])
  # @article.update(first_name: params[:student][:first_name], last_name: params[:student][:last_name])
  @student.update(params.require(:post).permit(:title, :description))
  redirect_to student_path(@article)
end
```
So then... with `@student.update`, the purpose of needing to `require` the `post` model is because of this: The `params` traditionally look like this:
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

## Side Note on `form_for` and `form_tag`
You only want to use `form_tag` syntax when creating a new object, but use `form_for` syntax when working with an object in the database that already exists. Like the difference between a `form_tag` in `new.html.erb` and the `form_for` in `edit.html.erb`.
