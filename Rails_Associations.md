# Foreign Keys

Just like in Sinatra, foreign keys are applied to a migration table in `db/migrate`.
An exemple can be found below:

```ruby
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.string :title
      t.string :content
      t.integer :author_id
    end
  end
end
```

The foreign key here in `:author_id` which is responsible for assigning a post to an author. After all, an author `has_many` posts, but a post `belongs_to` an author.
This can be illustrated below:

```ruby
class Student < ActiveRecord::Base
  has_many :assignments
end
```

```ruby
class Assignment < ActiveRecord::Base
  belongs_to :student
end
```
