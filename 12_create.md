# Updating an item

We can begin by going to a URL that is typically used to create a new resource or item.

```
http://localhost:3000/questions/new
```

This results in a conflicting error:

```
ActiveRecord::RecordNotFound in QuestionsController#show

	def show
		question_id = params[:id]
		* @question = Question.find(question_id) *
	end
```

Unfortunately the routing rule we have for show has captured this URL:

```ruby
get 'questions/:id' => 'questions#show', as: :question
```

To fix this we need to define a new route for the create and place it above the 'show rule'

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  get 'questions' => 'questions#index'
  get 'questions/new' => 'questions#new',  as: :new_question
  get 'questions/:id' => 'questions#show', as: :question
  get 'questions/:id/edit' => 'questions#edit', as: :edit_question
  patch 'questions/:id' => 'questions#update'
```

After applying this we end up with the next predictable error:

```ruby
The action 'new' could not be found for QuestionsController
```

We need to add a new action to the controller called 'new'. This action needs to create a new Question that will be used by the 'new view'.

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

		question_params = params.require(:question).permit(:title, :body)
		@question.update(question_params)

		redirect_to question_path(@question)
	end

	def new
		@question = Question.new
	end

end
```

Saving this and reloading the browse results in a missing template error:

```
Missing template questions/new
```

Next up we create this template. This template is going to be a 'copy & paste' from the edit template:

```html
<h3>New Question</h3>

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

The generated HTML for this looks a bit different to the edit page:

```html
<form class="new_question" id="new_question" action="/questions" accept-charset="UTF-8" method="post"><input name="utf8" type="hidden" value="&#x2713;" /><input type="hidden" name="authenticity_token" value="Jwlj1vkt/biT0C2YyPTrjqi7yeIE1R296dGOKNeaqSYZg7IV3KctE7DiTS0wo8nxqlr7HGe276rjKDZ2d+pcnQ==" />
	<p>
		<label for="question_title">Title</label>
		<input type="text" name="question[title]" id="question_title" />
	</p>
	<p>
		<label for="question_body">Body</label>
	</p>
	<p>	
		<textarea cols="30" rows="10" name="question[body]" id="question_body">
</textarea>
	</p>
	<p>
		<input type="submit" name="commit" value="Create Question" />
	</p>
</form>

```
The form_for method realized that this time it is dealing with a new 'question' - as opposed to updating \ patching one. Therefore it produced HTML that is slightly different.

There is no hidden field with the name of '_method'. This is going to be a post and Rails will interpret it as a post.

The form will submit a post to '/questions'. This will result in yet another routing error. Let's fill out some data and see the error:

```
No route matches [POST] "/questions"
```

Note this is different to the GET "/questions" route we have defined, this is a POST.

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  get 'questions' => 'questions#index'
  get 'questions/new' => 'questions#new', as: :new_question
  get 'questions/:id' => 'questions#show', as: :question
  get 'questions/:id/edit' => 'questions#edit', as: :edit_question
  patch 'questions/:id' => 'questions#update'

  post 'questions' => 'questions#create'

```

Attempting to submit the form again results in this error:

```
The action 'create' could not be found for QuestionsController
```

Next we can create that action:

```ruby
# app/controllers/questions_controller.rb

...
	def new
		@question = Question.new
	end

	def create
		question_params = params.require(:question).permit(:title, :body)
		question = Question.create(question_params)
		redirect_to question_path(question)
	end
...
	
```

That should do the trick. It should create the Question and redirect to the Show page for that Question.


## Refactoring

We can clean up the code by removing some duplication:

Update the controller to look as follows

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

	private 

	def question_params
		params.require(:question).permit(:title, :body)
	end

end

```

## Cleaning up the Routes

The routes.rb file is growing fairly rapidly considering that we only have one type of item in the system.

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  get 'questions' => 'questions#index'
  get 'questions/new' => 'questions#new', as: :new_question
  get 'questions/:id' => 'questions#show', as: :question
  get 'questions/:id/edit' => 'questions#edit', as: :edit_question
  patch 'questions/:id' => 'questions#update'

  post 'questions' => 'questions#create'
```

We can see what routes we have by typing `rake routes` in the terminal:

```
$ rake routes
       Prefix Verb  URI Pattern                   Controller#Action
    questions GET   /questions(.:format)          questions#index
 new_question GET   /questions/new(.:format)      questions#new
     question GET   /questions/:id(.:format)      questions#show
edit_question GET   /questions/:id/edit(.:format) questions#edit
              PATCH /questions/:id(.:format)      questions#update
              POST  /questions(.:format)          questions#create
```

We have been following Rails conventions (which are basically REST conventions). This means that we can use a helper method in Rails to generate all of these route definitions and a bit more. Change routes.rb to the following:

```ruby
# app/config/routes.rb

Rails.application.routes.draw do
  resources :questions
```

Now running rake routes produces the following:

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

All of the routes we have been using have been generated at runtime. As we have been following conventions, these generated ones match the routes we had manually entered - as well as the helper methods. 

There are also a few more e.g. DELETE and PUT

PUT is synonymous with PATCH, it is there for backwards compatibility. DELETE is for deleting items. In a future session we will use the delete route. 

## Exercise

Make it possible to add items to explore