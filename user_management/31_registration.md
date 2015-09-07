# User Registration

In this section we add the functionality which allows a user to sign up or register with the site.

We will add users through the following URL: http://localhost:3000/users/new

At the moment this gives us the following error: 

```
No route matches [GET] "/users/new"
```

We can fix this by adding a new resource to the routes.rb file:

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  root 'questions#index'

  resources :questions do
    resources :answers
  end

  resources :users
```

Adding this leads to the next error:

```
uninitialized constant UsersController
```

We generate that controller to resolve this problem:

```
rails generate controller users new
```

Look in routes.rb and remove this line if needed:

```ruby
get 'users/new'
```
Rails helped a bit too much...

In the 'new' action of the Users controller we need to create a new user object.

```ruby
class UsersController < ApplicationController
  def new
  	@user = User.new
  end
end
``` 

In the Users 'new' view we can add a form that will allow a user to sign up:

```html
<!-- app/views/users/new.html.erb -->
<h3>Sign up</h3>


<%= form_for(@user) do |f| %>
	<% if @user.errors.any? %>
	  	<p>The form contains <%= pluralize(@user.errors.count, "error") %> </p>
		<ul>
		<% @user.errors.full_messages.each do | message | %>
			<li><%= message %></li>
		<% end %>
		</ul>
	<% end %>
	<p>
		<%= f.label :name %>
	</p>
	<p>
		<%= f.text_field :name %>
	</p>
	<p>
		<%= f.label :email %>
	</p>
	<p>
		<%= f.email_field :email %>
	</p>
	<p>
		<%= f.label :password %>
	</p>
	<p>
		<%= f.password_field :password %>
	</p>
	<p>
		<%= f.label :password_confirmation, "Confirmation" %>
	</p>
	<p>
		<%= f.password_field :password_confirmation %>
	</p>
	<p>
		<%= f.submit "Sign up", class: "btn btn-primary" %>
	</p>
<% end %>
```


Submitting the form gives the next error:

```
The action 'create' could not be found for UsersController
```

We can fix this be adding the 'create' action:

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
	def new
		@user = User.new
	end	

  	def create
		
		@user = User.new(user_params)
		
		if(@user.save)
			flash[:success] = "Welcome to Questionable!"
			redirect_to user_path(@user)
		else
			render :new
		end
	end

	private 

	def user_params
		params.require(:user).permit(:name, :email, :password, :password_confirmation)
	end
end

```

We have implemented the create action in a similar way to previous 'create' actions. This action will be extended in the next section to create a session so that a user can be 'signed in'.

A reminder about the password confirmation: "Rails 4 completely ignores the confirmation if you don't send it as a parameter so if you forget to whitelist the password_confirmation attribute, it will be ignored and your validations won't say a thing. Careful of this!" - This is a comment from Stack Overflow - http://stackoverflow.com/a/18863258/259477 

The white listing mentioned here is the `params.require ...` line which is used in the controllers.


What we have done should handle the error cases, where a user does not meet the validation requirements. Next we show the profile page.

## The profile page

If a user signs up successfully we can show the profile page. To do this we need to first add a 'show' action to the controller:

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
	def new
		@user = User.new
	end	

  	def create
		
		@user = User.new(user_params)
		
		if(@user.save)
			flash[:success] = "Welcome to Questionable!"
			redirect_to user_path(@user)
		else
			render :new
		end
	end

	def show
		@user = User.find(params[:id])
	end

	private 

	def user_params
		params.require(:user).permit(:name, :email, :password, :password_confirmation)
	end
end
```

We also need the 'show' view:

```html
<!-- app/views/users/show.html.erb -->
<h3>Profile</h3>

<table>
	<tr><td>Name:</td><td><%= @user.name %></td></tr>
	<tr><td>Email:</td><td><%= @user.email %></td></tr>
</table>
```


Reminder: We added the logic to display the flash message in application.html.erb:

```ruby
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
<head>
  <title>Questionable</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
	<body>
		<div class="container">
		    <% flash.each do |name, msg| -%>
		      <%= content_tag :div, msg, class: name %>
		    <% end -%>

			<%= yield %>
		</div>
	</body>
</html>
```

Users can now sign up to our site, the next step is to allow them to sign in. This will allow us to protect sections of the site so that only certain users can access restricted pages.

## Exercise

Provide the ability for users to Register with Explore.  
