# Creating nested resources

In this section we create a new resource which is nested in another resource. In the context of Questionable, we will be adding a new answer to an existing question.

## Adding a link

First we create a link on the Show page of a question. To accomplish this we will use the `link_to` methods and a router helper method. 

To figure out the name of the route helper I can go to `http://localhost:3000/rails/info/routes` or run `rake routes` which shows 

```
              Prefix Verb   URI Pattern                                        Controller#Action
    question_answers GET    /questions/:question_id/answers(.:format)          answers#index
                     POST   /questions/:question_id/answers(.:format)          answers#create
 new_question_answer GET    /questions/:question_id/answers/new(.:format)      answers#new
edit_question_answer GET    /questions/:question_id/answers/:id/edit(.:format) answers#edit
     question_answer GET    /questions/:question_id/answers/:id(.:format)      answers#show
                     PATCH  /questions/:question_id/answers/:id(.:format)      answers#update
                     PUT    /questions/:question_id/answers/:id(.:format)      answers#update
                     DELETE /questions/:question_id/answers/:id(.:format)      answers#destroy
           questions GET    /questions(.:format)                               questions#index
                     POST   /questions(.:format)                               questions#create
        new_question GET    /questions/new(.:format)                           questions#new
       edit_question GET    /questions/:id/edit(.:format)                      questions#edit
            question GET    /questions/:id(.:format)                           questions#show
                     PATCH  /questions/:id(.:format)                           questions#update
                     PUT    /questions/:id(.:format)                           questions#update
                     DELETE /questions/:id(.:format)                           questions#destroy

```

The router helper we need is `new_question_answer_path`.

Within the questions Show template we can add:

```ruby
<!-- app/views/questions/show.html.erb -->
<h3><%= @question.title %></h3>

<p><%= @question.body %></p>

<%= link_to "Add Answer", new_question_answer_path(@question) %>

```

Clicking on that link navigates to a page with the URL of `http://localhost:3000/questions/3/answers/new`. This displays the following error:

```
uninitialized constant AnswersController
```

This means we need to generate the AnswersController. 

## Adding the controller and view

Based on the URL above (and our goal of creating a new answer) we know that we are going to need a `new` action. Therefore we can include this action when generating the controller.

```
$ rails generate controller answers new
      create  app/controllers/answers_controller.rb
       route  get 'answers/new'
      invoke  erb
      create    app/views/answers
      create    app/views/answers/new.html.erb
      invoke  test_unit
      create    test/controllers/answers_controller_test.rb
      invoke  helper
      create    app/helpers/answers_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/answers.coffee
      invoke    scss
      create      app/assets/stylesheets/answers.scss
```

The last two files are coffee script (JavaScript) and SCSS (“Sassy CSS" for styling). We will get to those later in the course. 


The AnswersController has already been created with an empty `new` action. 

```ruby
# app/controllers/answers_controller.rb
class AnswersController < ApplicationController
  def new
  end
end
```
There is also a view template that has been created
```html
<!-- app/views/answers/new.html.erb -->
<h1>Answers#new</h1>
<p>Find me in app/views/answers/new.html.erb</p>

```

Clicking on the `Add Answer` link now shows this 'new answer' view.

Next we need to customize the controller and view.

Within the AnswersController we need to obtain the question that this answer will be associated with. This question id is actually held in the params object. This is because of the routing rule we are using:

```
              Prefix Verb   URI Pattern                                        Controller#Action
 new_question_answer GET    /questions/:question_id/answers/new(.:format)      answers#new
```

The URI Pattern will capture the question id and store it in the params object with the key of `question_id`. 

You may recall that the new action of the QuestionController looks like this:

```ruby
	# app/controllers/questions_controller.rb
	def new
		@question = Question.new
	end
```

We need to do something similar here, however the Answer needs to be associated with the Question. We do that with the following code:

```
# app/controllers/answers_controller.rb
class AnswersController < ApplicationController
  def new
  	@question = Question.find(params[:question_id])
  	@answer = @question.answers.new
  end
end
```
This sets two variables for the view to use, `@question` and `@answer`. Creating '@answer' this way, means it already has the `@answer.question_id` set appropriately.

Before fleshing out the view, we can do a sanity test:

```html
<!-- app/views/answers/new.html.erb -->
<h3>New answer for: <%= @question.title %></h3>
```

This should display the correct information.

Next we need to build the form.

## A form for a nested resource

This is going to be very similar to the form we created before, only this time it is for a nested resource or nested model.

```html
<!-- app/views/answers/new.html.erb -->
<h3>New answer for: <%= @question.title %></h3>


<%= form_for([@question, @answer]) do | f | %>

<% end %>

```

Notice the line: `form_for([@question, @answer])`.

Passing an array into form_for causes a nested URL to be created in the HTML

```html
<form class="new_answer" id="new_answer" action="/questions/3/answers" accept-charset="UTF-8" method="post">
```

The complete form looks as follows:

```html
<!-- app/views/answers/new.html.erb -->
<h3>New answer for: <%= @question.title %></h3>


<%= form_for([@question, @answer]) do | f | %>
	<p>
		<%= f.label :body, "Please enter your answer" %>
	</p>
	<p>	
		<%= f.text_area :body, cols: 30, rows: 10 %>
	</p>
	<p>
		<%= f.submit %>
	</p>

<% end %>

```

Clicking the 'Create Answer' button takes us to the next error:

```
The action 'create' could not be found for AnswersController
```

This indicates that we need to modify the Answers controller.

## Fleshing out the controller

The create method looks as follows:

```ruby
# app/controllers/answers_controller.rb
class AnswersController < ApplicationController
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



