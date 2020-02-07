# Lifecycle Methods

These are methods that work to apply changes to code before it is saved to the database. These main methods are known as `before_save` and `before_validation`. Here is a rule of thumb: **Whenever you are modifying an attribute of the model, use `before_validation`. If you are doing some other action, then use `before_save`.**

```ruby
class Post < ActiveRecord::Base

  belongs_to :author
  validate :is_title_case

  before_validation :make_title_case

  private

  def is_title_case
    if title.split.any?{|w|w[0].upcase != w[0]}
      errors.add(:title, "Title must be in title case")
    end
  end

  def make_title_case
    self.title = self.title.titlecase
  end
end
```
 `before_validation` is used to execute the private method before validation occurs. `before_save` is called after validation occurs. So Rails goes `is valid?` and just... "Nope! Stop!", and never makes it to `before_save`. Thankfully, `before_validation` happens before `is_valid?` in the background and executes our code the way we need it to.
