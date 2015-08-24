# Delete

In this section we delete a resource, in the case of Questionable it is a 'question'

Running rake routes we can see that there is already a route defined:

```
$ rake routes
       Prefix Verb   URI Pattern                   Controller#Action
    questions GET    /questions(.:format)          questions#index
              POST   /questions(.:format)          questions#create
 new_question GET    /questions/new(.:format)      questions#new
edit_question GET    /questions/:id/edit(.:format) questions#edit
     question GET    /questions/:id(.:format)      questions#show
              PATCH  /questions/:id(.:format)      questions#update
              PUT    /questions/:id(.:format)      questions#update
              DELETE /questions/:id(.:format)      questions#destroy

```

This list indicates that the helper method for DELETE is 'question_path'.

We can add this ling to the 'show page'

```html
<!-- app/views/questions/show.html.erb -->
<h3><%= @question.title %></h3>

<p><%= @question.body %></p>

<%= link_to "Back to Questions", questions_path %>|
<%= link_to "Edit", edit_question_path(@question) %>|
<%= link_to "Delete", question_path(@question), method: :delete, data: {confirm: "Would you like to delete this?"} %>
```

Note this causes the following HTML to be generated:

```html
<a data-confirm="Would you like to delete this?" rel="nofollow" data-method="delete" href="/questions/4">Delete</a>

```

This causes the link to perform a 'DELETE'. In addition to this, there is now a confirmation pop-up.

Following the link through we end up with:

```
The action 'destroy' could not be found for QuestionsController
```

Next we add the destroy action which redirects to the index page:

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

	def index
		@questions = Question.all
	end

	def show
		@question = Question.find(params[:id])
	end

	def edit
		@question = Question.find(params[:id])
	end

	def update
		@question = Question.find(params[:id])
		@question.update(question_params)
		redirect_to question_path(@question)
	end

	def new
		@question = Question.new
	end

	def create
		question = Question.create(question_params)
		redirect_to question_path(question)
	end

	def destroy 
		question = Question.find(params[:id])
		question.destroy 
		redirect_to questions_url
	end

	private 

	def question_params
		params.require(:question).permit(:title, :body)
	end

end

```

## Exercise 

Modify the Explore so that users can delete resources.
