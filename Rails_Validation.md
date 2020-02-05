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


# Custom Validators

There are three ways to implement custom validators, with examples in Section 6 of the Rails Guide.

Of the three, #validate is the simplest. If your validation needs become more complex, consult the documentation. For most validations, though, the following method should be good enough.

1. Create a new directory in app called validators. Because most Rails developers don't need to write custom validation, this directory is not created by default like models or controllers.

2. Identify the ActiveRecord attribute you want to validate. Is it the email or the last_name on the Person class, for example?

3. Create a new file in the app/validators directory of the form attribute (from the previous step) + _validator.rb. So in the case of validating an attribute called email, create a file app/validators/email_validator.rb

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
