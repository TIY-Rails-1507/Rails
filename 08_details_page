# Routes part 1

The next objective is to show a details page for one item, in the case of Questionable this would be one question. 

## Starting from the URL...

Following a RESTful style architecture the URL to use is: 

http://localhost:3000/questions/1

Using that in the bowser results in an error:

```
No route matches [GET] "/questions/1"
```

This is because we have not defined a route. Looking in app/config/routes.rb we can see an example of a route that is similar to what we need.

```ruby
  # Example of regular route:
  #   get 'products/:id' => 'catalog#view'

```

Based off of that example, the required route is:
```ruby
  get 'questions' => 'questions#index'
  get 'questions/:id' => 'questions#show' # <-- new route
```
Note that :id is a variable (placeholder) field.

Adding that route and refreshing the browser leads to the next error:

```
The action 'show' could not be found for QuestionsController
```

This indicates we need to add the 'show' action to the QuestionsController

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
	def index
		@questions = Question.all
	end

	def show
	end
end
```

Just as the index action loads all the questions and assigns them to an instance variable. The show action needs to find the question with the ID from the URL. Recall that the URL is of the form: /questions/1

In controllers, we have access to a params hash. This contains any parameters from the URL. 

As we declared the route with: ```get 'questions/:id' => 'questions#show'```, the params hash contains :ID. 

This enables us to access the value we need:

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
	def index
		@questions = Question.all
	end

	def show
		@question = Question.find(params[:id])
	end
end
```

Adding that and refreshing the browser, leads to the next error: 

```
Missing template questions/show
``` 

Now we can add the missing template. We can use embedded Ruby to display some of the details of the question:

```html
<!-- app/views/questions/show.html.erb -->

<h3><%= @question.title %></h3>

<p><%= @question.body %></p>
```html

Now refreshing the browser serves up a page with the details of the requested Question.





