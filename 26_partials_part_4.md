# Partials - part 4 

This section provides a brief recap of the topics on partials previously covered.

## The task
In this section we want to display answers from a partial so that they can be used in both the answers index page as well as a question's show page.

## The answers index page

This page should show all of the answers for a question. 

This page will be shown at the following URL: http://localhost:3000/questions/1/answers

### The answers controller

The answers controller needs an index action which fetches all the answers for a question:

```ruby
# app/controllers/answers_controller.rb
class AnswersController < ApplicationController
	
	def index
		@question = Question.find(params[:question_id])
		@answers = @question.answers
	end

	def new
	  	@question = Question.find(params[:question_id])
	  	@answer = @question.answers.new
	end

	def create
	  	@question = Question.find(params[:question_id])
	  	@answer = @question.answers.new(answer_params)
	  	@question.save
	    redirect_to question_path(@question)
	end

	private 

	def answer_params
		params.require(:answer).permit(:body)
	end
end
```

### The answers index page

Let's start by creating the 'answers partial'. A good name for it is '_answer.html.erb'.
* all partials start with an underscore
* calling is this means that the 'built in' variable will have a name of `answer`.

This partial is designed to display a single answer:

```html
<!-- app/views/answers/_answer.html.erb -->
<div>
	<p>
		<%= answer.body %>
		<br/>
		Answered <%= time_ago_in_words(answer.created_at) %> ago
	</p>
</div>
```

For information on `time_ago_in_words` see http://apidock.com/rails/ActionView/Helpers/DateHelper/time_ago_in_words

As the naming conventions have been followed, the view can use the following rendering shortcut:

```html
<!-- app/views/answers/index.html.erb -->
<h3>Answers</h3>

<p><%= @question.title %></p>

<p>Answers</p>

<%= render @answers %>
```

This works because
* the models in @answers are of type 'answer'
* this means that Rails will display them using the partial '_answer.html.erb'
* Rails applies that template to each answer

Next we can reuse the partial in the Questions show page


### The questions model

### The questions controller

### The questions show page

#### Further reading
http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials

Specifically: 
* Search the page for: heterogeneous collection
* 3.4.6 Local Variables
* 3.4.7 Spacer Templates


## Exercises

Use a partial with a collection within Explore.