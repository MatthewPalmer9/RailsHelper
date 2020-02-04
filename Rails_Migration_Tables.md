# Proper Structure for Migration Tables
```ruby
class CreateArtists < ActiveRecord::Migration
  def change
    create_table :artists do |t|
      t.string :name
      t.text :bio

      t.timestamps null: false
    end
  end
end
```

Be sure to include `t.timestamps null: false` so that ActiveRecord can keep track of when records are created/stored in our database.
