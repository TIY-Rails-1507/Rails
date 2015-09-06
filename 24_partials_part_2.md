# Partials - part 2

In this section we will dig deeper into partials and see how they can be used to divide a view up into manageable sections. This will give us the added benefit of removing duplication in the application.

## Background

At the moment, Questionable shows 3 tables on the Index view for a Question. One table shows all the questions, one table shows questions which don't have answers, and the third table shows all the questions which have answers.

To get this behavior the following custom query methods were added to the Question model:

```ruby
# app/models/question.rb
class Question < ActiveRecord::Base
	has_many :answers

	validates :title, presence: true
	validates :body, length: { minimum: 10 }

	def self.without_answers
		includes(:answers).where(:answers => { :id => nil })
	end

	def self.with_answers
		includes(:answers).where.not(:answers => { :id => nil })
	end
end
```
Ref: http://stackoverflow.com/questions/18082096/rails-4-scope-to-find-parents-with-no-children/18082147#comment52450047_18082147


The controller can then use these custom queries:
```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

	def index
		@all_questions = Question.all
		@answered_questions = Question.with_answers
		@unanswered_questions = Question.without_answers
	end
```

The index view has the following:

```html
<h3>Questions</h3>

<p><%= link_to "Add Question", new_question_path %> </p>

<table border="1">
  <caption>All Questions</caption>
  <tr>
    <th>Title</th>
    <th>Description</th>
    <th>Answers</th>
    <th></th>    
  </tr>
  <% @all_questions.each do | question | %>  
  <tr>
    <td><%= link_to(question.title,  question_path(question)) %></td>
    <td><%= format_body(question) %></td>
    <td><%= link_to(question.answers.count, question_answers_path(question)) %></td>
    <td>
      <%= link_to "Show", question_path(question) %> | 
      <%= link_to "Edit", edit_question_path(question) %> | 
      <%= link_to "Delete", question_path(question), method: :delete, data: {confirm: "Would you like to delete this?"} %>

    </td>
  </tr>
  <% end %>
</table>

<table border="1">
  <caption>Unanswered Questions</caption>
  <tr>
    <th>Title</th>
    <th>Description</th>
    <th>Answers</th>
    <th></th>    
  </tr>
  <% @unanswered_questions.each do | question | %>  
  <tr>
    <td><%= link_to(question.title,  question_path(question)) %></td>
    <td><%= format_body(question) %></td>
    <td><%= link_to(question.answers.count, question_answers_path(question)) %></td>
    <td>
      <%= link_to "Show", question_path(question) %> | 
      <%= link_to "Edit", edit_question_path(question) %> | 
      <%= link_to "Delete", question_path(question), method: :delete, data: {confirm: "Would you like to delete this?"} %>

    </td>
  </tr>
  <% end %>
</table>

<table border="1">
  <caption>Answered Questions</caption>
  <tr>
    <th>Title</th>
    <th>Description</th>
    <th>Answers</th>
    <th></th>    
  </tr>
  <% @answered_questions.each do | question | %>  
  <tr>
    <td><%= link_to(question.title,  question_path(question)) %></td>
    <td><%= format_body(question) %></td>
    <td><%= link_to(question.answers.count, question_answers_path(question)) %></td>
    <td>
      <%= link_to "Show", question_path(question) %> | 
      <%= link_to "Edit", edit_question_path(question) %> | 
      <%= link_to "Delete", question_path(question), method: :delete, data: {confirm: "Would you like to delete this?"} %>

    </td>
  </tr>
  <% end %>
</table>

```

Notice the repetition in each table. That is what we are going to remove in this section.

## Introducing a partial

First we will extract a partial to handle the table for All Questions. To do this we create a partial in app/views/questions called '_questions_table.html.erb'. We will copy the 'All Questions' table into it:

```html
<!-- app/views/questions/_questions_table.html.erb -->
<table border="1">
  <caption>All Questions</caption>
  <tr>
    <th>Title</th>
    <th>Description</th>
    <th>Answers</th>
    <th></th>    
  </tr>
  <% @all_questions.each do | question | %>  
  <tr>
    <td><%= link_to(question.title,  question_path(question)) %></td>
    <td><%= format_body(question) %></td>
    <td><%= link_to(question.answers.count, question_answers_path(question)) %></td>
    <td>
      <%= link_to "Show", question_path(question) %> | 
      <%= link_to "Edit", edit_question_path(question) %> | 
      <%= link_to "Delete", question_path(question), method: :delete, data: {confirm: "Would you like to delete this?"} %>

    </td>
  </tr>
  <% end %>
</table>

```

As with all partials the name of the file starts with an underscore. To render this partial, we use `<%= render "questions_table" %>`. This is without the underscore or file extensions. 

Both the view and partial are in the same folder. If they were not we would need to prefix the name of the partial with the folder path e.g. `render "shared/questions_table"`.

As we copied the 'all questions' table into the partial, we simply need to render the partial in the index page and the user will see the same page:

```html
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

<p><%= link_to "Add Question", new_question_path %> </p>

<%= render "questions_table" %>


<table border="1">
  <caption>Unanswered Questions</caption>
  <tr>
    <th>Title</th>
    <th>Description</th>
    <th>Answers</th>
    <th></th>    
  </tr>
  ...
  ...
  ...
```

The partial makes use of `@all_questions`. This works as it was in scope in the view that rendered the partial. While this works for the All Questions table, we can't use this partial as it is for the Unanswered Questions table - it needs `@unanswered_questions`. To pass different 'questions' to the partial we need to make the partial more flexible. 

## Making the partial more flexible

In order to reuse the partial for all 3 tables (or lists of questions) we need to pass a 'local variable' into the partial. To do this we render the partial as follows:

```html
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

<p><%= link_to "Add Question", new_question_path %> </p>

<%= render partial: "questions_table", locals: { questions: @all_questions} %>
...
...
...
``` 

We can then access this local variable in the partial as follows:

```html
<!-- app/views/questions/_questions_table.html.erb -->

<table border="1">
  <caption>All Questions</caption>
  <tr>
    <th>Title</th>
    <th>Description</th>
    <th>Answers</th>
    <th></th>    
  </tr>
  <% questions.each do | question | %>  
  <tr>
    <td><%= link_to(question.title,  question_path(question)) %></td>
    <td><%= format_body(question) %></td>
    <td><%= link_to(question.answers.count, question_answers_path(question)) %></td>
    <td>
      <%= link_to "Show", question_path(question) %> | 
      <%= link_to "Edit", edit_question_path(question) %> | 
      <%= link_to "Delete", question_path(question), method: :delete, data: {confirm: "Would you like to delete this?"} %>

    </td>
  </tr>
  <% end %>
</table>
```
The key line being: `<% questions.each do | question | %>`

Note that it does not use the `@` symbol, as it is considered a local variable.

Now we can try and use the partial to replace the Unanswered Questions:

```html
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

<p><%= link_to "Add Question", new_question_path %> </p>

<%= render partial: "questions_table", locals: { questions: @all_questions} %>
<%= render partial: "questions_table", locals: { questions: @unanswered_questions} %>

<table border="1">
  <caption>Answered Questions</caption>
  <tr>
    <th>Title</th>
    ...
    ...
    ...
```

This got us most of the way, the only problem is the hard coded caption, which now shows 'All Questions' on the All Questions table as well as the Unanswered Questions table. To fix this we can pass in an additional parameter:

```html
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

<p><%= link_to "Add Question", new_question_path %> </p>

<%= render partial: "questions_table", locals: { questions: @all_questions, caption: "All Questions" } %>
<%= render partial: "questions_table", locals: { questions: @unanswered_questions, caption: "Unanswered Questions" } %>
...
...
...
```

Then update the partial to make use of this new variable:

```html
<!-- app/views/questions/_questions_table.html.erb -->

<table border="1">
  <caption><%= caption %></caption>
  <tr>
    <th>Title</th>
    <th>Description</th>
    <th>Answers</th>
    <th></th>    
  </tr>
  <% questions.each do | question | %>  
  <tr>
    <td><%= link_to(question.title,  question_path(question)) %></td>
    <td><%= format_body(question) %></td>
    <td><%= link_to(question.answers.count, question_answers_path(question)) %></td>
    <td>
      <%= link_to "Show", question_path(question) %> | 
      <%= link_to "Edit", edit_question_path(question) %> | 
      <%= link_to "Delete", question_path(question), method: :delete, data: {confirm: "Would you like to delete this?"} %>

    </td>
  </tr>
  <% end %>
</table>
```

We can now use the partial for all three tables:

```html
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

<p><%= link_to "Add Question", new_question_path %> </p>

<%= render partial: "questions_table", locals: { questions: @all_questions, caption: "All Questions" } %>
<%= render partial: "questions_table", locals: { questions: @unanswered_questions, caption: "Unanswerd Questions" } %>
<%= render partial: "questions_table", locals: { questions: @answered_questions, caption: "Answered Questions" } %>
```

The view is a lot smaller and cleaner now. Next we explore different ways of using partials.

## Using the built in variable

There is a 'built in' or implicit variable that we can use with any partial. To use this we pass a variable in using a key of `:object` and then the partial can access this variable by using the name of the partial. 

To use this approach, the view changes to this:

```html
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

<p><%= link_to "Add Question", new_question_path %> </p>

<%= render partial: "questions_table", object: @all_questions, locals: { caption: "All Questions" } %>
<%= render partial: "questions_table", object: @unanswered_questions, locals: { caption: "Unanswerd Questions" } %>
<%= render partial: "questions_table", object: @answered_questions, locals: { caption: "Answered Questions" } %>
```

And the partial now accesses 'object' by using the name of the partial:

```html
<!-- app/views/questions/_questions_table.html.erb -->

<table border="1">
  <caption><%= caption %></caption>
  <tr>
    <th>Title</th>
    <th>Description</th>
    <th>Answers</th>
    <th></th>    
  </tr>
  <% questions_table.each do | question | %>  
  <tr>
    <td><%= link_to(question.title,  question_path(question)) %></td>
    <td><%= format_body(question) %></td>
...
...
...
```

To make the code a bit more readable we can change the file name of the partial from `_questions_table.html.erb` to `_questions.html.erb`. Then the code in the partial becomes:

```html
<!-- app/views/questions/_questions.html.erb -->

<table border="1">
  <caption><%= caption %></caption>
  <tr>
    <th>Title</th>
    <th>Description</th>
    <th>Answers</th>
    <th></th>    
  </tr>
  <% questions.each do | question | %>  
  <tr>
  ...
  ...
  ...
```

The view needs to be updated to use the new name of the partial:

```html
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

<p><%= link_to "Add Question", new_question_path %> </p>

<%= render partial: "questions", object: @all_questions, locals: { caption: "All Questions" } %>
<%= render partial: "questions", object: @unanswered_questions, locals: { caption: "Unanswered Questions" } %>
<%= render partial: "questions", object: @answered_questions, locals: { caption: "Answered Questions" } %>

```

## Exercise 

Use partials to remove the duplicate tables in Explore












