# RUBY ON RAILS CRUD (Create, Read, Update, Destroy)

## How To Set a CREATE path without mapping to create.html.erb
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
