# Basic Usage of Validations

For more detailed information not covered in this repository on ActiveRecord Validations with Rails, please visit https://guides.rubyonrails.org/active_record_validations.html

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true
end

Person.create(name: "John Doe").valid? # => true
Person.create(name: nil).valid? # => false
```

`#validates` is our Swiss Army knife for validations. It takes two arguments: the first is the name
of the attribute we want to validate, and the second is a hash of options that will include the
details of how to validate it. In this example, the options hash is `{ presence: true }`, which implements
the most basic form of validation, preventing the object from being saved if its `name` attribute is empty.

# Displaying Validation Errors in Views
```ruby
<% if @article.errors.any? %>
  <div id="error_explanation">
    <h2>
      <%= pluralize(@article.errors.count, "error") %>
      prohibited this article from being saved:
    </h2>

    <ul>
    <% @article.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```

This constructs more complete-looking sentences from the more terse messages available in `errors.messages`.

# Other Built-in Validators
## Length
`length` is one of the most versatile

```ruby
class Person < ActiveRecord::Base
  validates :name, length: { minimum: 2 }
  validates :bio, length: { maximum: 500 }
  validates :password, length: { in: 6..20 }
  validates :registration_number, length: { is: 6 }
end
```

**Side note: The above syntax is Ruby "poetry mode". If we stopped using it, it would look like the following: **

```ruby
class Person < ActiveRecord::Base
  validates(:name, { :length => { :minimum => 2 } })
  validates(:bio, { :length => { :maximum => 500 } })
  validates(:password, { :length => { :in => 6..20 } })
  validates(:registration_number, { :length => { :is => 6 } })
end
```

## Uniqueness
```ruby
class Account < ActiveRecord::Base
  validates :email, uniqueness: true
end
```

`uniqueness` will prevent any account in this case from being created with the same email as another already-existing account.

# Inclusion & Custom Validation
```ruby
class Post < ActiveRecord::Base
  validates :title, presence: true
  validates :content, length: {:minimum => 250}
  validates :summary, length: {:maximum => 250}
  validates :category, inclusion: {in: ["Fiction", "Non-Fiction"]}
  validate :clickbait

  def clickbait
    if !title.nil? && (!title.include?("You Won't" || "Secret" || "Top" || "Guess"))
      errors.add(:clickbait, "That is not a clickbait worthy title. Try again.")
    end
  end
end
```

# Validations in Controller Actions
```ruby
# app/controllers/posts_controller.rb

  def create
    # Create a brand new, unsaved, not-yet-validated Post object from the form.
    @post = Post.new(post_params)

    # Run the validations WITHOUT attempting to save to the database, returning
    # true if the Post is valid, and false if it's not.
    if @post.valid?
      # If--and only if--the post is valid, do what we usually do.
      @post.save
      # This returns a status_code of 302, which instructs the browser to
      # perform a NEW REQUEST! (AKA: throw @post away and let the show action
      # worry about re-reading it from the database)
      redirect_to post_path(@post)
    else
      # If the post is invalid, hold on to @post, because it is now full of
      # useful error messages, and re-render the :new page, which knows how
      # to display them alongside the user's entries.
      render :new
    end
  end
```

`render` can be instructed to render the templates from other actions. In the above code, since we want the `:new` template from the same controller, we don't have to specify anything except the template name.

Here is an example of what `def update` looks like with controller validations:
```ruby
  def update
    if @post.update(post_params)

      redirect_to post_path(@post)
    else
      render :edit
    end
  end

  private

  def post_params
    params.permit(:category, :content, :title)
  end
```

Remember: **redirects incur a new page load**. When we redirect after validation failure, we **lose** the instance of `@post` that has feedback (messages for the user) in its `errors` attribute.

Another way to differentiate redirects is this:

If you hit Refresh after a redirect/page load, your browser resubmits the `GET` request without complaint.

If you hit Refresh after rendering on a form submit, your browser gives you a popup to confirm that you want to resubmit form data with the `POST` request.

# Validations with Forms (`form_tag` and `form_for`)
```ruby
<!-- app/views/people/new.html.erb //-->

<%= form_tag "/people" do %>
  Name: <%= text_field_tag "name", @person.name %><br>
  Email: <%= text_field_tag "email", @person.email %>
  <%= submit_tag "Create Person" %>
<% end %>
```

The second argument to text_field_tag, as with most form tag helpers, is the "default" value.
Below is an example of what the raw HTML would look like.

```ruby
Name: <input type="text" name="name" id="name" value="Jane Developer" /><br />
Email: <input type="text" name="email" id="email" value="jane@developers.fake" />
```


# Custom Validators

There are three ways to implement custom validators, with examples in Section 6 of the Rails Guide.

Of the three, #validate is the simplest. If your validation needs become more complex, consult the documentation. For most validations, though, the following method should be good enough.

1. Create a new directory in app called validators. Because most Rails developers don't need to write custom validation, this directory is not created by default like models or controllers.

2. Identify the ActiveRecord attribute you want to validate. Is it the email or the last_name on the Person class, for example?

3. Create a new file in the app/validators directory of the form attribute (from the previous step) + `_validator.rb`. So in the case of validating an attribute called email, create a file app/validators/email_validator.rb

4. Inside the new file, define the class. The class name should match the file name of the file, but "Camel-Cased." So email_validator should be class EmailValidator. The class should inherit from ActiveModel::Validator

5. The validator class must have one instance method, #validate. This method will receive one argument typically called record.

6. Inside of #validate, you'll be able to get properties from record and determine whether it is invalid. If the record is invalid, push (<<) to record.errors[:attribute] e.g. record.errors[:email] a String which is a message that you want to display that describes why the message is not valid.

7. Lastly, in the implementation of the class being validated e.g. Person, add:

  8. An include of ActiveModel::Validations

  9. The helper call: validates_with (className). In our example we'd put, validates_with EmailValidator (see step 4, above)
  For more detailed information on ActiveRecord Validations with Rails, please visit https://guides.rubyonrails.org/active_record_validations.html

The results should be the following:

```ruby
class EmailValidator < ActiveModel::Validator
  def validate(record)
    unless record.email.match?(/railsgit.com/)
      record.errors[:name] << "We're only allowed to have people who work for the company in the database!"
    end
  end
end
```
```ruby
class Person
  include ActiveModel::Validations
  validates_with EmailValidator
end
```

Here we validate that all email addresses are in the `railsgit.com` domain.

# Displaying All Errors with `errors.full_messages`

When a model fails validation, its `errors` attribute is filled with information about what went wrong. Rails creates an `ActiveModel::Errors` object to carry this information.

The simplest way to show errors is to just spit them all out at the top of the form by iterating over `@person.errors.full_messages.` But first, we'll have to check whether there are errors to display with `@person.errors.any?`.
```ruby
<% if @person.errors.any? %>
  <div id="error_explanation">
    <h2>There were some errors:</h2>
    <ul>
      <% @person.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```

The whole picture comes down to this code snippet below:

```ruby
<%= form_tag("/people") do %>
  <% if @person.errors.any? %>
    <div id="error_explanation">
      <h2>There were some errors:</h2>
      <ul>
        <% @person.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>


  <div class="field<%= ' field_with_errors' if @person.errors[:name].any? %>">
    <%= label_tag "name", "Name" %>
    <%= text_field_tag "name", @person.name %>
  </div>

  <div class="field<%= ' field_with_errors' if @person.errors[:email].any? %>">
    <%= label_tag "email", "Email" %>
    <%= text_field_tag "email", @person.email %>
  </div>

  <%= submit_tag "Create" %>
<% end %>
```

**It can get tedious dealing with `form_tag`. It is encouraged to use `form_for` soon to be explained below.**
