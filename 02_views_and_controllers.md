# Views and Controllers 

These notes are related to 'Questionable', you need to apply the concepts to 'Explore'.


## Display a list of questions

The goal is to list a the current set of questions within the system. 

Doing a 'GET' on http://localhost:3000/questions

Should display all the questions in the system.

To achieve this I need to do the following:

1. Create a route 
2. Create a controller
3. Create a view


### Create a route

The URL which should show the questions is: 

http://localhost:3000/questions


Currently that shows an error.


```No route matches [GET] "/questions"```

Routes are defined in ```config/routes.rb```

The route needed is:
```
get 'questions' => 'questions#index'
```

After a browser refresh, the error changes to:

```
uninitialized constant QuestionsController
```

To generate a controller, open the terminal at the root directory and enter:

```
rails generate controller questions
```

There is now a new file at: app/controllers/questions_controler.rb

After a browser refresh, the following error is shown:

```
The action 'index' could not be found for QuestionsController
```

We are missing an action (method) on the new QuestionsController.

Change this:
```ruby
# app/controllers/questions_controler.rb
class QuestionsController < ApplicationController
end
```
To:

```ruby
# app/controllers/questions_controler.rb
class QuestionsController < ApplicationController
  def index
  end
end
```

Refreshing the browser, results in another issue:

```
Missing template questions/index, application/index
```

Next, we need to create the view. This can be done as an erb file (embedded ruby). 

```html
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

These are the current questions:
<ul>
  <li>Is Pluto a planet?</li>
  <li>How do you make a website?</li>
</ul>
```

Refreshing the browser shows the page!

Note the conventions:
* The route specified that the action was 'index'
* We Controller simply had an empty action named index
* Rails then looked up the view in the conventional path

### Exercise 
Make the Explore app display the names of a few hotels.


## Using ERB templates

We are currently using static data, the same content will be rendered each time a visitor comes to the site. To make the page more dynamic we can embed Ruby into the ERB templates.

To do this, we add an instance member variable in the controller e.g.

```ruby
# app/controllers/questions_controler.rb
class QuestionsController < ApplicationController
  def index
      @questions = ['Is Pluto a planet?', 'Why am I here?', 'Who am I?']
  end
end
``` 

Then in the view we use embedded Ruby:


```html
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

These are the current questions:
<ul>
<% @questions.each do |question| %>
  <li><%= question %></li>
<% end %>  
</ul>
```

### Exercise 
Make the view in the Explore app use embedded Ruby in the view.
