# Partial Querying

Partials work to help query a database from a search bar. In order to accomplish this, we need to focus on a few areas in our application. The `controllers`, the `models` and the `views`.

Let's start with the controller. In this example, we will focus on a `students_controller.rb`. If we wanted to do our querying on our `index.html.erb` page under the `students` `views`, we need to write some `params` variables in our `Student#index`.

```ruby
class Student < ActiveRecord::Base
  def index
    @students = Student.search(params[:query])
    render 'index'
  end
end
```

*Note: The `.search` is manually defined in our `Student` model. This is what it looks like:*

```ruby
  def self.search(query)
    if query.present?
      where('NAME like ?', "%#{query}%")
    else
      self.all
    end
  end
```

Here, we are asking the `Student#index` controller to recognize `@students` to equal whatever the search `params` is... given that the `:query` exists in our database. Notice we are rendering the `index` (which is whatever we have written in our `index.html.erb`). In this case, this is what our `index.html.erb` looks like:

```erb
<%= form_tag students_path, method: :get do %>
  <p>
    <%= text_field_tag :query, params[:query] %>
    <%= submit_tag "Search", name: nil %>
  </p>
<% end %>

<% @students.each do |student| %>
  <%= render partial: 'student', locals: {student: student} %>
<% end %>
```

*Note: The `:query` `params` in the controller is set here inside of our `index.html.erb` by the `text_field_tag :query, params[:query]`. As a refresher, our `params` is passed into the `Student#index` and renders our `index` again.*

Located below our field tags are where we render our partial with locals. (**Please see Rails_Partials_With_Locals.md if how this works isn't clear**). This partial is located at `views/students/_student.html.erb`. Example:

```erb
<ul>
  <li>
    Name: <%= student.name %>
  </li>
  <li>
    Birthday: <%= student.birthday.strftime("%m/%d/%Y")  %>
  </li>
  <li>
    Hometown: <%= student.hometown.capitalize %>
  </li>
</ul>
```

## Conclusion
Now we know how to query using partials with locals. It's a complicated process, and remembering how to write them is **hard**. Referencing either this guide or the Ruby Docs is perfectly acceptable. The purpose here is incorporating things like search bars into our websites while keeping our code DRY.
