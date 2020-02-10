# Creating People

How do we write our Person form? We don't want to require our user to first create an Address, then create that Person. That's annoying. We want a single form for a Person containing several slots for their addresses.

We don't want to manually write setters like Song#artist_name= to find or create an Artist and connect them to the song.

That won't work here, because an address contains more than one field. In the Artist case we were just doing the name. With Address, it's "structured data". All that really means is it has multiple fields attached to it. When we build a form for it, the form will send a different key for each field in each address. This can get a bit unwieldy so we generally try to group a hash within the params hash, which makes things much neater. Spoiler alert: Rails has a way to send this across as a hash.

The complete params object for creating a Person will look like the following. Using "0" and "1" as keys can seem a bit odd, but it makes everything else work moving forward. This hash is now more versatile. You can access nested values the standard way, with params[:person][:addresses_attributes]["0"] returning all of the information about the first address at 33 West 26th St.

```ruby
{
  :person => {
    :name => "Avi",
    :addresses_attributes => {
      "0" => {
        :street_address_1 => "33 West 26th St",
        :street_address_2 => "Apt 2B",
        :city => "New York",
        :state => "NY",
        :zipcode => "10010",
        :address_type => "Work"
      },
      "1" => {
        :street_address_1 => "11 Broadway",
        :street_address_2 => "2nd Floor",
        :city => "New York",
        :state => "NY",
        :zipcode => "10004",
        :address_type => "Home"
      }
    }
  }
}
```

We can plug this information in by typing `bundle exec rails console` and entering the following:
```ruby
new_person = Person.new
new_person.addresses_attributes={
  "0"=>{
    "street_address_1"=>"33 West 26",
    "street_address_2"=>"Floor 2",
    "city"=>"NYC",
    "state"=>"NY",
    "zipcode"=>"10004",
    "address_type"=>"work1"
    },
  "1"=>{
    "street_address_1"=>"11 Broadway",
    "street_address_2"=>"Suite 260",
    "city"=>"NYC",
    "state"=>"NY",
    "zipcode"=>"10004",
    "address_type"=>"work2"
    }
  }
  new_person.save
```

This will also allow us to then use `new_person.addresses` and get access to all of the has information, but only if your `person.rb` in `models` has the following:

```ruby
class Person < ActiveRecord::Base
  has_many :addresses
  accepts_nested_attributes_for :addresses
end  
```

## Now, we can apply our form!

```ruby
# app/views/people/new.html.erb

<%= form_for @person do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %><br>

  <%= f.fields_for :addresses do |addr| %>
    <%= addr.label :street_address_1 %>
    <%= addr.text_field :street_address_1 %><br>

    <%= addr.label :street_address_2 %>
    <%= addr.text_field :street_address_2 %><br>

    <%= addr.label :city %>
    <%= addr.text_field :city %><br>

    <%= addr.label :state %>
    <%= addr.text_field :state %><br>

    <%= addr.label :zipcode %>
    <%= addr.text_field :zipcode %><br>

    <%= addr.label :address_type %>
    <%= addr.text_field :address_type %><br>
  <% end %>

  <%= f.submit %>
<% end %>
```

The `fields_for` line gives something nice and English-y. In that block are the fields for the addresses. Love Rails.
Load up the page, and see the majestic beauty of what you and Rails have written together. But wait... Nothing is there?

Here is why...

# Creating Stubs

We're asking Rails to generate `fields_for` each of the Person's addresses. However, when we first create a `Person`, they have no addresses. Just like `f.text_field :name` will have nothing in the text field if there is no name, `f.fields_for :addresses` will have no address fields if there are no addresses.

We'll take the most straightforward way out: when we create a `Person` in the `PeopleController`, we'll add two empty addresses to fill out. The final controller looks like this:

```ruby
class PeopleController < ApplicationController
  def new
    @person = Person.new
    @person.addresses.build(address_type: 'work')
    @person.addresses.build(address_type: 'home')
  end

  def create
    person = Person.create(person_params)
    redirect_to people_path
  end

  def index
    @people = Person.all
  end

  private

  def person_params
    params.require(:person).permit(:name)
  end
end
```

Now, refresh the page, and you'll see two lovely address forms. Try to hit submit, and it isn't going to work. One last hurdle. We have new `params` keys, which means we need to modify our `person_params` method to accept them. Your `person_params` method should now look like this:

```ruby
def person_params
  params.require(:person).permit(
    :name,
    addresses_attributes: [
      :street_address_1,
      :street_address_2,
      :city,
      :state,
      :zipcode,
      :address_type
    ]
  )
end
```

## When Using `accepts_nested_attributes_for` Is A Bad Idea
In the cases above, it made sense to use `accepts_nested_attributes_for` since two People can live at the same address. However, let's say we have a database that needs to keep record of artists without them being duplicated. We would want `Artist` rows to be unique so that, say, something like `Artist.find_by(name: 'Bullet For My Valentine')` works properly. So, the alternative method is to apply this code into your `models`.

```ruby
# app/models/song.rb

class Song < ActiveRecord::Base
  def artist_attributes=(artist)
    self.artist = Artist.find_or_create_by(name: artist[:name])
    self.artist.update(artist)
  end
end
```
