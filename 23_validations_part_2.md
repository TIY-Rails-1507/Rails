# Validations

Now that we have validations in the model, we can make use of them in the Controller and View.

## Controller

You may recall that `question.save` returned false when the model was invalid, indicating that it did not save. `question.save` returns true when a save has been carried out successfully.

We can make use of this within the Controller. 

The Questions Controller gets changed from:

```ruby
	def create
		question = Question.create(question_params)
		redirect_to question_path(question)
	end
```

To:

```ruby
	def create
		@question = Question.new(question_params)
		
		if(@question.save)
			redirect_to question_path(@question)
		else
			render :new
		end
	end
```

If the validation fails, we call `render :new`. This will render the new template without calling the 'new' action on the controller. The 'new' template is passed the @question back again so that it renders any fields which were populated by the user.

Next we need to show validation errors on the form.

## Updating the view

To display the validation errors we can update the form partial:

```html
<!-- app/views/questions/_form.html.erb -->

<%= form_for(@question) do |f| %>
	<%= @question.errors.full_messages.to_sentence %>
	<p>
		<%= f.label :title %>
		<%= f.text_field :title, autofocus: true %>
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

`@question.errors.full_messages.to_sentence` displays the errors in a readable format.

Another option is to create a list of the errors:

```html

<%= form_for(@question) do |f| %>
	<p>The form contains <%= pluralize(@question.errors.count, "error") %> </p>
	<ul>
	<% @question.errors.full_messages.each do | message | %>
		<li><%= message %></li>
	<% end %>
	</ul>
	<p>
		<%= f.label :title %>
		<%= f.text_field :title, autofocus: true %>
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

This works fine for Create, however the controller has not been setup to use validations in the update action.

## Back to the controller

We need to change the update action from:

```ruby
	def update
		@question = Question.find(params[:id])
		@question.update(question_params)
		flash[:notice] = "The question was updated"
		redirect_to question_path(@question)
	end
```

To: 
```ruby
	def update
		@question = Question.find(params[:id])
		if @question.update(question_params)
			flash[:notice] = "The question was updated"
			redirect_to question_path(@question)
		else
			render :edit
		end
	end
```

There is something very subtle going on in the view when there is a validation error. When we do get a validation error, Rails wraps the input field, and corresponding label, in a div tag which which has a class set to to `field_with_errors`.

This HTML looks like this:

```html
<p>
		<div class="field_with_errors"><label for="question_body">Body</label></div>
	</p>
	<p>	
		<div class="field_with_errors"><textarea cols="30" rows="10" name="question[body]" id="question_body">
Abc</textarea></div>
	</p>
``` 

We can add an entry to our CSS file which could make use of this to style the error messages. 

### OK... let's add some color

To highlight the error add the following to the end of app/assets/stylesheets/application.css

```html

.field_with_errors > label  {
    color: red;
}

.field_with_errors > textarea,
.field_with_errors > input {
    color: red;
    border-style: solid;
    border-width: 2px;
}

```

Add that to the end, do not remove any line from this file.

Please ask if you would like to know exactly how that CSS works...


## Extra

If you prefer to use a `span` and not a `div` in the error messages, read this: http://apidock.com/rails/ActionView/Helpers/ActiveRecordHelper/error_messages_for#119-Overriding-the-default-div-class-fieldWithErrors-

## Exercise

Make the Explore app show validation errors - the CSS bit is optional.

Please don't add CSS to each element on your page, we will be using Bootstrap to help style the app.

