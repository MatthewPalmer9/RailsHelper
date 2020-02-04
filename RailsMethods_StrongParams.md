# How To Use link_to To Access /object_path/:id
```ruby
<div>
  <% @students.each do |student| %>
    <div><%= link_to student.to_s, student_path(student) %></div>
  <% end %>
</div>
```

The most important thing to understand here is that `link_to` is going to create
an `<a>` tag with a href attribute equal to the student id by using `student_path(student)`.

```ruby
# Example Output:
<div>
    <div><a href="/students/1">Scotty Ruth</a></div>
</div>
```

# How to Use Strong Params
Rails needs to be secure between routes. That's why we need to `permit` what parameters are passed into a database.
If it exists there, `config.action_controller.permit_all_parameters = true` needs to be removed from `config/application.rb`.
then, we need to update a couple controller methods...
```ruby
# app/controllers/posts_controller.rb
  # ...
  def create
    @post = Post.new(post_params(:title, :description))
    @post.save
    redirect_to post_path(@post)
  end

  def update
    @post = Post.find(params[:id])
    @post.update(post_params(:title))
    redirect_to post_path(@post)
  end

  private


  # We pass the permitted fields in as *args;
  # this keeps `post_params` pretty dry while
  # still allowing slightly different behavior
  # depending on the controller action. This
  # should come after the other methods

  def post_params(*args)
    params.require(:post).permit(*args)
  end
```
