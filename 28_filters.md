# Filters

"Filters are methods that are run before, after or "around" a controller action."

- http://guides.rubyonrails.org/action_controller_overview.html#filters

We can use this remove duplication in the Answers controller.


## Applying a filter

We are going to change the Answers controller, not the usual Questions controller.

Currently the Answers Controller looks as follows:

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

We have duplication with the following line: `@question = Question.find(params[:question_id])`

We can remove that by using a `before_action`. The `before_action` will get called once before each action. We can set an instance member variable that can be used in all actions:

```ruby
# app/controllers/answers_controller.rb
class AnswersController < ApplicationController
	
	before_action :set_question

	def index
		@answers = @question.answers
	end

	def new
	  	@answer = @question.answers.new
	end

	def create
	  	@answer = @question.answers.new(answer_params)
	  	@question.save
	    redirect_to question_path(@question)
	end

	private 

	def set_question
		@question = Question.find(params[:question_id])
	end

	def answer_params
		params.require(:answer).permit(:body)
	end
end
```

If you only want to apply a filter to some actions you can use `:only` or `:except`. For example:

```ruby
before_action :set_question, only: [:new, :create]

```

The above was just an example, applying that would cause errors.

## Warning: don't load data in a filter

Using a `before_action` is a good way to apply access control. You can ensure that a user is logged in before they can 'use' any actions. We will cover this in upcoming weeks.

In the example above we used a `before_action` to load data. Not everyone agrees with this. It does remove duplication, however it adds complexity. This is a post from 'crafting ruby' which argues this case - http://craftingruby.com/posts/2015/05/31/dont-use-before-action-to-load-data.html

According to that post, we would be better of removing the duplication by using plain old Ruby methods to load data:

```ruby
# app/controllers/answers_controller.rb
class AnswersController < ApplicationController

	def index
		@answers = find_question.answers
	end

	def new
	  	@answer = find_question.answers.new
	end

	def create
		@question = find_question
	  	@answer = @question.answers.new(answer_params)
	  	@question.save
	    redirect_to question_path(@question)
	end

	private 

	def find_question
		@question = Question.find(params[:question_id])
	end

	def answer_params
		params.require(:answer).permit(:body)
	end
end

```