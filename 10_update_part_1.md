# Updating data - part 1

Typically a user can Create, Read, Update and Delete data. These actions are known as CRUD operations. 

In Questionable the user can already read data. In this section we will add the functionality to update data.

## Begin with a URL...

The conventional way to edit in Rails is to take a user to a URL similar to this: 

```
http://localhost:3000/questions/1/edit
```

That should show a form which allows the user to edit a question with an ID of 1. At the moment it shows a `No route matches [GET] "/questions/1/edit"` error.

We need to add another route to routes.rb:
```
get 'questions/:id/edit' => 'questions#edit', as: :edit_question
```

That specifies that a URL matching the pattern on the left, should call the `edit` action on the `QuestionsController`. That route has a name of `edit_question`. We can use this name in the helper methods if we wish e.g.

```html
<%= link_to "Edit", edit_question_path(@question) %>
```

Refreshing the browser leads to the following error:
```
The action 'edit' could not be found for QuestionsController
```

We can add this action to the `QuestionsController`.

While we are doing this, we can fetch the question we want to edit.

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

	def index
		@questions = Question.all
	end

	def show
		question_id = params[:id]
		@question = Question.find(question_id)
	end

	def edit
		question_id = params[:id]
		@question = Question.find(question_id)
	end

end
```



Refreshing the browser shows a missing template error:
```
Missing template questions/edit
```

We can create a basic view, to make sure the controller is working as expected:

```html
<h3><%= @question.title %></h3>

```
This can be tested by passing through different IDs in the URL.

```
http://localhost:3000/questions/1/edit
http://localhost:3000/questions/2/edit
http://localhost:3000/questions/3/edit
```

Next, we require an HTML form which will allow a user to edit the Question.

The HTML we wish to end up with is displayed next. Note we don't have to type this, as Rails has methods to help generate this HTML:

```html
<form  method="post">
	<p>
		<label for="question_title">The title goes here</label>
		<input type="text" value="Milk?" name="question[title]" id="question_title" />
	</p>
	<p>
		<label for="question_body">The body will go here</label>
	</p>
	<p>	
		<textarea cols="30" rows="10" name="question[body]" id="question_body"></textarea>
	</p>
	<p>
		<input type="submit" name="commit" value="Update Question" />
	</p>
</form>

```


To do this in ERB we use the following helper methods:

```html
<!-- app/views/questions/edit.html.erb -->
<h3><%= @question.title %></h3>

<%= form_for(@question) do |f| %>
	<p>
		<%= f.label :title %>
		<%= f.text_field :title %>
	</p>
	<p>
		<%= f.label :body %>
	</p>
	<p>	
		<%= f.text_area :body, cols: 30, rows: 10 %>
	</p>
	<p>
		<%= f.submit %>
	</p>
<% end %>

``` 
The form_for methods is a powerful construct. Notice how the form object `f` is used in subsequent lines to build up the form.

If there was a number or a price, we could have used:

```
f.number_field :price
```

It is worth looking into the datetime_select method - http://api.rubyonrails.org/classes/ActionView/Helpers/DateHelper.html#method-i-datetime_select

Before we wire up the form, let's have a checkpoint exercise

### Exercise 

Add an edit page for the hotels (or any resource you like).

Use `rake routes` to see the name of the edit route.

Use that route name to add an edit link in the details page of a hotel.

Use that route name to add edit links to each hotel in the index page. It would be a good idea to add another column to the table which had the text 'edit' and that linked to the edit page for that hotel.


