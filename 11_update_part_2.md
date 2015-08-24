# Updating data - part 2

Next up we need to submit the form data so that the server can save the update. As Rails encourages a RESTful design.

To see more, have a read about REST in the file: A01_Update_REST_Rails.md


## A 'fake' PATCH request

To do an Update in Rails we need to do a PATCH request.

Most browsers support GET and POST however not all support DELETE, PATCH (or PUT).

Rails has a work around for this by using a hidden field. Next we will return to the Questionable app to see the details.

The Edit view currently has this code:

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

If we look at the source code within the browser we see the following HTML.

```html
<form class="edit_question" id="edit_question_3" action="/questions/3" accept-charset="UTF-8" method="post"><input name="utf8" type="hidden" value="&#x2713;" /><input type="hidden" name="_method" value="patch" /><input type="hidden" name="authenticity_token" value="cuas2XtDJGJRWTweuWqRJmDEpBdg/Q/jibxeyjc8zwxMbH0aXsn0yXJrXKtBPbNZYiWW6QOe/fSDReaUl0w6tw==" />
	<p>
		<label for="question_title">Title</label>
		<input type="text" value="Milk?" name="question[title]" id="question_title" />
	</p>
	<p>
		<label for="question_body">Body</label>
	</p>
	<p>	
		<textarea cols="30" rows="10" name="question[body]" id="question_body">
Is milk good for me?</textarea>
	</p>
	<p>
		<input type="submit" name="commit" value="Update Question" />
	</p>
</form>
```
First we can notice the form tag declaration:

```html
<form class="edit_question" id="edit_question_3" action="/questions/3" accept-charset="UTF-8" method="post">
```
At the end of the tag we see `method="post"`. This means that when we hit the submit button this form will do a post to "/questions/3". 

All of the data in the form will be sent to the server in the body of the request. This is made available in the params array.

In the generated HTML we see this line:

```html
<input type="hidden" name="_method" value="patch" />
```
This is a hidden field which means that it won't be visible by the user. However the Rails framework can use this and treat this request as if it was a PATCH request.

In summary, even though this request will technically be a POST, Rails will treat it as a PATCH. By convention, a PATCH request means that we are performing an update. 

## Implementing the Update

If we open the browser to an edit page e.g.

http://localhost:3000/questions/3/edit

We can click the 'Update Question' button to submit the form. We expect Rails to view this as a PATCH request to "/questions/3". We don't have a route for that, so after clicking the button we get this error:

```
No route matches [PATCH] "/questions/3"
```

To fix this we add a route:

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  get 'questions' => 'questions#index'
  get 'questions/:id' => 'questions#show', as: :question
  get 'questions/:id/edit' => 'questions#edit', as: :edit_question
  patch 'questions/:id' => 'questions#update'
```

There is no need to name this route as we don't have a need for the URL helper methods.

Trying to submit the form again we get:

```
The action 'update' could not be found for QuestionsController
```

This means we have route but no action in the controller. This 'update' action needs fetch the form data from the params hash as well as update the Question in the database.

All of the question data is stored in the params hash, under the key of 'question'. Rails is able to do this because the HTML has fields such as this:

```html
<input type="text" value="Milk?" name="question[title]" id="question_title" />
```

Rails can looks at the name field and use that to build a nested hash that we can use within the controller. 

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

	def update
		question_id = params[:id]
		@question = Question.find(question_id)
		@question.update(params[:question])
	end

end
```

This results in this error:

```
ActiveModel::ForbiddenAttributesError in QuestionsController#update
```

This is due to a security implementation in Rails. As the developer we need to specify which attributes we are allowing the user to update. This prevents a hacker from changing fields that they are not meant to e.g. the ID

To white list these editable fields we need to do the following:

```ruby
# app/controllers/questions_controller.rb

	def update
		question_id = params[:id]
		@question = Question.find(question_id)

		question_params = params.require(:question).permit(:title, :body)
		@question.update(question_params)
	end

```

Here is a terse explanation of that code: http://stackoverflow.com/a/18426829/259477  - make sure you read the comment as well.


After this change, clicking the update question button will result in another error:

```
Missing template questions/update
```
This is because we don't have a view that would pair with the update action. However we can check that the model data was updated in the database. You can test this by doing one of these:
* Look at the index page
* Look at the details page for one item
* Use the rails console to query the database

## Redirect to the details page

Given that we already have a 'show' view which shows the details of a question, we can simply redirect to that.

```ruby
	def update
		question_id = params[:id]
		@question = Question.find(question_id)

		question_params = params.require(:question).permit(:title, :body)
		@question.update(question_params)

		redirect_to question_path(@question)
	end
```

This redirect, caused the browser to make another GET request to the URL which would show the details of a question e.g. /questions/1

This should now show the updated question, reusing the 'show' template.

## Exercise
Make the hotels (or any resource) editable in the Explore application.



