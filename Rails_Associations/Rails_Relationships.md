# The `has_many :objects, through: :another_object` Relationship

This relationship is particularly a difficult concept to grasp. For this example, let's use Taxis, Rides, and Passengers to help illustrate this concept. An important thing to understand about these types of relationships are that they work with **foreign keys** in your database.

For our example, here are what your migration files should look like:

```ruby
class CreateTaxis < ActiveRecord::Migration
  def change
    create_table :taxis do |t|
      t.timestamps null: false
    end
  end
end
```

```ruby
class CreateRides < ActiveRecord::Migration
  def change
    create_table :rides do |t|
      t.integer :passenger_id
      t.integer :taxi_id
      t.timestamps null: false
    end
  end
end
```

```ruby
class CreatePassengers < ActiveRecord::Migration
  def change
    create_table :passengers do |t|
      t.timestamps null: false
    end
  end
end
```

Each of the above tables already have a primary key thanks to ActiveRecord automatically assigning them. However, our `:rides` table contains two foreign keys `:passenger_id` and `:taxi_id`. We can now create our relationships in `app/models`.

```ruby
class Taxi < ActiveRecord::Base
  has_many :rides
  has_many :passengers, through: :rides
end
```

```ruby
class Ride < ActiveRecord::Base
  belongs_to :taxi
  belongs_to :passenger
end
```

```ruby
class Passenger < ActiveRecord::Base
  has_many :rides
  has_many :taxis, through: :rides
end
```

You can also think of this relationship visually like this: `Taxi -< Ride >- Passenger`
