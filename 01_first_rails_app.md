
# First Rails App - almost...


## Generate the skeleton template 

In your ```dev``` directory type the following:

```rails new explore```

Navigate into the root directory of the explore application

```cd explore```

The explore directory is what we will call the projects rails root directory, or simply the root directory.

Open this root directory in your editor. 

Make the following change to ensure the database is connected when we load the Rails console:

```ruby
# config/application.rb

class Application < Rails::Application

	console do
	  ActiveRecord::Base.connection
	end
end
```
We will come back to this later, however if you wish to know more see here: http://stackoverflow.com/a/20474666/259477

After saving ```aplication.rb``` we can run the application.


## Launch Explore

Within the projects rails root directory (explore) type:

```
rails server
```

This should launch a WEBrick server. If it does not, then try:

```
rails server webrick
```

Now the app is running and we can view it in a browser on one of these URLs
* http://0.0.0.0:3000
* http://localhost:3000

Play around with the links and see what useful references are pointed to e.g. Rails API

Try to find the currently configured database adapter.

Leave the server running and open a new console\terminal window to continue working...

## What just happened?

We installed rails and created our first app. Rails generated a load of files for us. The main folders we should note are:
* app
  * where the main application code is written 
* app/view
  * The HTML templates will be stored here
* app/controller
  * The application orchestration code happens here
  * It co-ordinates the view and the models
* app/models
  * This is where the domain models are found
  * Generally speaking these represent the data in the application
  * They apply business rules and logic
* config
  * the main configuration is done here e.g. choosing a database

If you are still keen on more detail see: http://www.tutorialspoint.com/ruby-on-rails/rails-directory-structure.htm 



