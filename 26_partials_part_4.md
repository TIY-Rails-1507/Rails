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

### The questions show page

As the questions show view is in a different folder to the partial '_answer.html.erb', we need to prefix the name of the partial with the folder the partial is in:

```html
<!-- app/views/questions/show.html.erb -->
<h3><%= @question.title %></h3>

<p><%= @question.body %></p>

<%= link_to "Add Answer", new_question_answer_path(@question) %> | 
<%= link_to "Back to Questions", questions_path %> | 
<%= link_to "Edit", edit_question_path(@question) %> | 
<%= link_to "Delete", question_path(@question), method: :delete, data: {confirm: "Would you like to delete this?"} %>

<p>Answers</p>

<%= render partial: "answers/answer", collection: @question.answers %>
```

#### Further reading
http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials

Specifically: 
* Search the page for: heterogeneous collection
* 3.4.6 Local Variables
* 3.4.7 Spacer Templates (see below)


## Exercises

Use a partial with a collection within Explore.


## Bonus

### Spacer templates

The views are looking a bit crowded, to help resolve this we can use spacer templates. This is a way of displaying another template between the 'main data templates'. This is best expressed in an example. 

Let's assume we want to display some kind of 'break' between the items in the questions lists as well as lists of answers.

As this partial will be shared, we can put it in a new folder called 'shared'.

```html
<!-- app/views/shared/_break.html.erb -->
<hr>
```
This is simply a horizontal ruler. Having it in one place is useful should we choose to change or style this 'break marker'.

To use this partial we can use the `spacer_template:`. This would change the Answers index page from: 

```ruby
<!-- app/views/answers/index.html.erb -->
<h3>Answers</h3>

<p><%= @question.title %></p>

<p>Answers</p>

<%= render @answers %>
```
To: 

```ruby
<!-- app/views/answers/index.html.erb -->
<h3>Answers</h3>

<p><%= @question.title %></p>

<p>Answers</p>

<%= render partial: @answers, spacer_template: "shared/break" %>
```

This will insert a horizontal line between each answer. Note that no data is passed to the spacer template (break).

To use this in the Questions index we need to change the questions partial from:

```html
<!-- app/views/questions/_questions.html.erb -->

<div>
	<h3><%= caption %></h3> 
	<%= render questions %>  
</div>

```

To:

```html
<!-- app/views/questions/_questions.html.erb -->

<div>
	<h3><%= caption %></h3> 
	<%= render partial: questions, spacer_template: "shared/break" %>  
</div>
```
