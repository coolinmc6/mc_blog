# README

This app is a simple blog application following the tutorial by [Mackenzie Child](https://mackenziechild.me/) in his
12-apps-in-12-weeks series.  This video for the tutorial is located here: [Week 2: How to build a blog in Rails 4](https://mackenziechild.me/12-in-12/2/) and here: [Youtube](https://youtu.be/BI_VnnOLSKY?list=PL23ZvcdS3XPLNdRYB_QyomQsShx59tpc-).

####Things to look up later:
- Hipster ipsum
- strong parameters

####Running Questions
- What are the basics of a create method?  How is the create method different than the 'new' method?
- How does params[:id] work?  What is it doing?


####~0:00 - 4:15
- create new app
- initialize git repo

####~4:20
```shell
rails generate controller posts
```
- Notice all the items that are created when you build a an app:
```shell
create  app/controllers/posts_controller.rb  <= NEW FILE
invoke  erb
create    app/views/posts   <= NEW FILE
invoke  test_unit
create    test/controllers/posts_controller_test.rb   <= NEW FILE
invoke  helper
create    app/helpers/posts_helper.rb   <= NEW FILE
invoke    test_unit
invoke  assets
invoke    coffee
create      app/assets/javascripts/posts.coffee   <= NEW FILE
invoke    scss
create      app/assets/stylesheets/posts.scss   <= NEW FILE
```
So with a controller, you get:
  - the controller itself (posts_controller.rb) which is pretty much blank:
```ruby
class PostsController < ApplicationController
end
```
  * a view for posts, test file, helperfile, scss file and .coffee file.

Add the following to config/routes.rb:
```ruby
	resources :posts
	root "posts#index"
```
*Notice that when you run the server and try to go to localhost:3000 (the root), there is an error because we
don't yet have a view!  SOOOO....when we created a controller, we did NOT get a view with it, so we'll need
to create one ourselves.
####~6:40
- Create a 'new' action in the posts_controller.rb file.  Notice how if we go to ...3000/posts/new we get the same 
error as before in that there is no template.
- Create a 'new' template for posts: new.html.erb

####~7:30 - Use form_for
```html
<%= form_for :post, url: posts_path do |f| %>
	<p>
		<%= f.label :title %>
		<%= f.text_field :title %>
	</p>
	<p>
		<%= f.label :body %>
		<%= f.text_area :body %>
	</p>
	<p>
		<%= f.submit %>	
	</p>
```
####~9:40
- By this time, we have created a simple view to receive posts BUT we don't yet have a model that can actually
save the data.  That's what we need to do now...
```shell
rails g model Post title:string body:text
```
- In order to save our posts, we have to define a 'create' method
- A 'create' method pretty much has to just create the post (@post = Post.new) and save it (@post.save).  BUT....there's
more to it than that...
  * as a security feature, Rails require that we specify the parameters that we are updating in something called "strong
parameters"
  * strong parameters are defined in the Post controller (posts_controller.rb) and are 'private' meaning that they [FILL IN]
* this is what the create and strong parameters look like:
```ruby
def create
	@post = Post.new(post_params)
	@post.save

	redirect_to @post
end

private

	def post_params
		params.require(:post).permit(:title, :body)
	end
```
####~13:05
- After completing the above steps, when we try to create a new post, we get this error: **The action 'show' could 
not be found for PostsController**.  And why is that?  We have told the application in the 'create' action
to redirect the user to '@post' which means that we want to *show* them the post.
  *...I believe this is where Rails is actively looking for the 'show' view
- So we *should* have two next steps: build a 'show' action and 'show' template

####~15:00
- So we've built the view BUT it will not work because we actually haven't yet defined '@post'.  But what about the 
'@post' in the create method?  I don't think that is available to 'show' because it is defined inside 'create'.
*We've created the 'show' method now and here it is:
```ruby
def show
	@post = Post.find(params[:id])
end
```
- If ever struggling as to what you want the action to do, just think about for a minute.  What am I trying to
accomplish with the 'show' method?  I want to *show* the user their new post.  And how do I do that? I search
the database for a post with the given id.
  * I need to learn more about the params[:id] in Rails

####~16:00
* At this point, as much I want to clean-up the view and add a link to go back, what would I be going back to?
Right now, nothing is set-up to show ALL the posts...so I need to create a way to show all posts. 
  *and to start, we need to update our 'index' action
  *and display everything in reverse order

```ruby
def index
	@post = Post.all.order('created_at DESC')
end
```

####~18:45
* At this point, I can create posts, I can view all of them, and I can view individual ones.  Notice the code to show
the date in Ruby...I need to learn it:
```ruby
<%= post.created_at.strftime("%b %d, %Y") %>
```
* We still can't edit or delete yet

####~19:45 - Styling
```shell
git checkout -b styling_v2
```
* We brought in basic styling
* Notice how we import the normalize partial in the application.scss file
```css
@import 'normalize';
```

####~28:40
* Google fonts in Rails:
```html
<%= stylesheet_link_tag 'application', 'http://fonts.googleapis.com/css?family=Raleway:400,700' %>
```

####~35:00 - Validations
```shell
git checkout -b validations
```
* Although I would like to explain these validations out later, here is the validations for the Post model:
```ruby
class Post < ApplicationRecord
	validates :title, presence: true, length: { minimum: 5}
	validates :body, presence: true
end
```
* We also updated the posts_controller.rb.  First, we updated the new action which was blank and then updated the
create action.  Here is the new code:
```ruby
def new
	@post = Post.new
end

def create
	@post = Post.new(post_params)

	if @post.save
		redirect_to @post
	else
		render 'new'
	end
end
```
* We updated the create action with an if-statement.  This is what is happening:
  1. if @post.save is successful (it successfully saves to the database), then redirect to the post they just wrote
    * this is important because we added validations to the post model that requires a title length minimum of 5
    and the body must have something there.
  2. if @post.save is unsucessful (it fails one of the validations and CANNOT be saved to the database), we are are
  going to *render* 'new'.  
    * This is important because it saves the person's data, it allows them to avoid re-doing an entire post

####~40:50
* Update posts_controller.rb
* How does the update action work?  Why is it params[:post]?

####~42:30
* Good spot to see how he removes most of the show.html.erb file and puts it into a partial _form.html.erb


####~44:20 - Add Delete action


```ruby
def destroy
	@post = Post.find(params[:id])
	@post.destroy

	redirect_to root
end
```
* This is how he deleted a post and I must break into:
```ruby
<%= link_to 'Destroy', post_path(@post), method: :delete, data: { confirm: "Are you sure?"} %>
```
* This was located in the show.html.erb (so when you go into a particular post) and I need to learn what each of 
these things are. 

####~47:15 - Add Comments
```shell
rails g model Comment name:string body:text post:references
rails db:migrate
```
* This is what I am doing:
  * I am generating a model called Comment
  * It has a column called 'name' (string) and a column called 'body' (text)
  * It belongs_to Post.  Again...what this means is that Rails automatically puts that line in there:
```ruby
class Comment < ApplicationRecord
		belongs_to :post
end
```
  * Rails does NOT add the corresponding line into the post.rb file (the Post model)
```ruby
class Post < ApplicationRecord
	has_many :comments
	validates :title, presence: true, length: { minimum: 5}
	validates :body, presence: true
end
```
  * And lastly, because it created a migration, we run 'rails db:migrate'

* We then have to update the config/routes.rb file to make nested routes:
```ruby
resources :posts do 
	resources :comments
end
```
* Generate Comments controller
```shell
rails g controller Comments
```
* Write the create action in the Comments controller

####58:00
* By this time we have created the _comment and _form partials (in the views/comments folder)
* We have written the destroy action for Comments
* And we now need to make it so that when a Post is destroyed, the associated comments are destroyed as well.  That can be done quite easily by doing the following:
```ruby
# old version
has_many :comments
# new version
has_many :comments, dependent: :destroy
```
####59:00 - Pages Controller
```shell
rails g controller pages
```
* We want to create an About page so we create an 'about' action and update the routes.rb file:
```ruby
get '/about', to: 'pages#about'
```
  * What this does is allow us to simply go to /about and not /pages/about.  Notice the syntax...I need to learn this.
  * If we try to go to /about, it fails because we do not yet have a template...

####1:03:00 - Add Users
* Revisit this, we add the Devise gem here
```shell
rails g devise:install
```
```shell
rails g devise:views
```
```shell
rails g devise User
rails db:migrate
```
* He hides a lot of buttons using the if-user-is-signed-in logic as follows: 
```ruby
<% if user_signed_in? %>
	| <%= link_to 'Edit', edit_post_path(@post) %>
	| <%= link_to 'Destroy', post_path(@post), method: :delete, data: { confirm: "Are you sure?"} %>
<% end %>
```

* This is how I logout:
```html
<button class="button"><%= link_to "Logout", destroy_user_session_path, method: :delete %></button>
```











