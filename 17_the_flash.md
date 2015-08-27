# Flash!

While we are not that concerned about the look of the website at the moment, it would still be nice to give feedback to the user when an item has been created, updated or deleted. 

To do this we use the 'flash' object. This is a hash like object that is available in the controllers and views.

As explained in the Rails guides, "The flash is a special part of the session which is cleared with each request. This means that values stored there will only be available in the next request, which is useful for passing error messages etc."

http://guides.rubyonrails.org/action_controller_overview.html#the-flash

While you can use any key for the flash message it is common to use :notice, :alert or the general purpose :flash.

Here is an example of the flash message being set in a controller:

```ruby
	def update
		@question = Question.find(params[:id])
		@question.update(question_params)
		flash[:notice] = "The question was updated"
		redirect_to question_path(@question)
	end
```

While this sets a key-value pair on the flash, we will still need to display this in a view. 

A useful place to add this is in app/views/application.html.erb. Placing it here means that it will be available to all pages.

```html
<!DOCTYPE html>
<html>
<head>
  <title>Questionable</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
	<body>
	    <%= flash[:notice] %>

		<%= yield %>

	</body>
</html>

``` 

It is often useful to place this in a div, as that will move other elements off of the same line:

```html
	<body>
	    <%= content_tag :div, flash[:notice], class: name %>

		<%= yield %>

	</body>
</html>

```

Another tactic is to loop through all the keys in the flash so that it will display all the messages:

```html
	<body>
	    <% flash.each do |name, msg| %>
	      <%= content_tag :div, msg, class: name %>
	    <% end %>

		<%= yield %>

	</body>
```

The way we have used the flash here is good for 99% of the time. However there are a few special conditions to be aware of when using the flash, these are mentioned within the rails guides: http://guides.rubyonrails.org/action_controller_overview.html#the-flash


Now try and make use of the flash in your application...
