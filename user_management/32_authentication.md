# Authentication

While we have added the functionality to create new users, we still need to create the sign in functionality.

In Rails it is common to maintain state between page requests with the use of cookies. Cooks are pieces of information which can be saved by the browser. The information contained in the cookie can be posted back with each request. This is known as a 'session'.

When a user logs in we want to start a session, this session can be destroyed when a user logs out. This will be managed via a Sessions controller.

## Sessions Controller - new

We can generate a new controller:

```
$ rails generate controller sessions new
```

Again this will generate a route which is not exactly as we want it, so delete:

```
get 'sessions/new'
```

Instead of that we will use a few named routes:

```ruby
# app/config/routes.rb

Rails.application.routes.draw do

  root 'questions#index'

  get    'login'   => 'sessions#new'
  post   'login'   => 'sessions#create'
  delete 'logout'  => 'sessions#destroy'

  resources :questions do
    resources :answers
  end

  resources :users
```

Take a moment to review those routes before continuing. 

Doing a get on 'http://localhost:3000/login' will show the 'new' view which Rails generated. Next we will turn that into a 'login form'.

## Login Form

This form is different from previous forms as we do not have a model. Therefore wont use `form_for` but instead we will use `form_tag`:

```html
<!-- app/views/sessions/new.html.erb -->
<h3>Login</h3>

<%= form_tag(login_path) do %>
	<p>
		<%= label_tag(:email) %>
	</p>
	<p>	
		<%= text_field_tag(:email) %>
	</p>
	<p>
		<%= label_tag(:password) %>
	</p>
	<p>
		<%= password_field_tag %>
	</p>
	<p>
		<%= submit_tag("Log in") %>
	</p>
<% end %>
``` 
Take note of the HTML that this produces by viewing the page source in the browser.

An alternative would have been to use `form_for(:session, url: login_path)`.

Submitting this form results in the following error:

```
The action 'create' could not be found for SessionsController
```

## Sessions Controller - create

Next up we need to find and authenticate the user. If the user exists, we will create a session. 

Let's start with the case where a user cannot be found:

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
	def new
	end

	def create
		user = User.find_by(email: params[:email].downcase)
		if user && user.authenticate(params[:password])
			# to come...
		else
			flash.now[:alert] = "Invalid email or password"	
			render 'new'
		end
	end
end
```
Take note of these lines:

```ruby
user = User.find_by(email: params[:email].downcase)
if user && user.authenticate(params[:password])
```

The if statement above uses the logical short-circuit. If no user is found with the provided email then `user` will be `nil`. This will evaluate to false and the rest of the if statement will not be evaluated. This avoids calling a method on a nil object.

You may have noticed that we are using `flash.now`. This is because the flash message lasts for one request. When we use a `redirect` it causes the browser to make a new request, the flash message is shown and expired. The browser does not perform a new request when we use `render`. This causes the flash message to hang around for one more request than is needed. Using `flash.now` avoids this problem. 

Feel free to experiment with this by removing the `.now`, submitting the form with invalid data, then perform a 'get' to another URL. You will see that the flash message remains for one more request.

### Creating a session

When a user logs in with valid credentials we need to create a session. This is done by using the `session` object. 

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
	def new
	end

	def create
		user = User.find_by(email: params[:email].downcase)
		if user && user.authenticate(params[:password])
			session[:user_id] = user.id
			redirect_to user_path(user)
		else
			flash.now[:alert] = "Invalid email or password"	
			render 'new'
		end
	end
end
```

The key line is: 

```ruby
session[:user_id] = user.id
```

This creates a temporary cookie within the users browser. It will last until the user closes the browser. It is also encrypted and can't be tampered with. 

We will be using that line in multiple places so it would be a good idea to add it to the Session Helper:

```ruby
# app/helpers/sessions_helper.rb
module SessionsHelper
	def log_in(user)
		session[:user_id] = user.id
	end
end
```

To make use of this helper in the Session Controller (and all other controllers) `include` the helper in the Application Controller

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  include SessionsHelper
end
```

This can now be used by the controller with:

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
	def new
	end

	def create
		user = User.find_by(email: params[:email].downcase)
		if user && user.authenticate(params[:password])
			log_in(user)
			redirect_to user_path(user)
		else
			flash.now[:alert] = "Invalid email or password"	
			render 'new'
		end
	end
end
```

We can reuse this `login` method and login after a successful sign up:

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
	def new
		@user = User.new
	end	

  	def create
		
		@user = User.new(user_params)
		
		if(@user.save)
			log_in(@user)
			flash[:success] = "Welcome to Questionable!"
			redirect_to user_path(@user)
		else
			render :new
		end
	end
...
...
...
```

Other convenient methods we could add to the helper is `current_user` and `logged_in?`:

```ruby
# app/helpers/sessions_helper.rb
module SessionsHelper
	def log_in(user)
		session[:user_id] = user.id
	end

	def current_user
    	@current_user ||= User.find_by(id: session[:user_id])
  	end

  	def logged_in?
    	!current_user.nil?
  	end
end
```

## Login, Logout & Sign up links

If the user is not signed in then we would like to show the login and sign up links. If the user is currently signed in then we want to show a logout link. It is a good idea to add this logic to the application.html.erb.

```html
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
		<% if logged_in? %>
			<%= link_to "Profile", user_path(current_user) %> | 
			<%= link_to "Log out", logout_path, method: "delete"  %>

		<% else %>
			<%= link_to "Log in", login_path %> | 
 			<%= link_to "Sign up", new_user_path %>
		<% end %>
		<div class="container">
		    <% flash.each do |name, msg| -%>
		      <%= content_tag :div, msg, class: name %>
		    <% end -%>

			<%= yield %>
		</div>
	</body>
</html>

```

Clicking logout results in the following error:

```
The action 'destroy' could not be found for SessionsController
```

## Logging Out 

Loggin out is simply a matter of deleting the session variable and clearing the 'cached' @current_user variable. We can add this in the session helper:

```ruby
# app/helpers/sessions_helper.rb
module SessionsHelper
	def log_in(user)
		session[:user_id] = user.id
	end

	def current_user
    	@current_user ||= User.find_by(id: session[:user_id])
  	end

  	def logged_in?
    	!current_user.nil?
  	end

	def log_out
		session.delete(:user_id)
		@current_user = nil
	end  	
end
```

We can then use this method in the controller:

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
	def new
	end

	def create
		user = User.find_by(email: params[:email].downcase)
		if user && user.authenticate(params[:password])
			log_in(user)
			redirect_to user_path(user)
		else
			flash.now[:alert] = "Invalid email or password"	
			render 'new'
		end
	end

	def destroy
	    log_out
	    redirect_to root_url
  	end
end

```

## Exercise 

Add the sign up, login and log out functionality to Explore. 

