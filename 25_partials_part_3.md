# Partials - part 3

## Updating the Questions partial

Let's assume that we have been asked to change the layout. Users don't like the table format and want a more vertical layout. To do this we can simply change the partial:

```html
<!-- app/views/questions/_questions.html.erb -->

<div>
	<h3><%= caption %></h3> 
	<% questions.each do | question | %> 
		<div> 
			<h4><%= question.title %></h4>
			<p><%= format_body(question) %></p>
			<p><%= link_to(pluralize(question.answers.count, "answer"), question_answers_path(question))  %></p>
			<p><%= link_to "Show", question_path(question) %> | <%= link_to "Edit", edit_question_path(question) %> |  <%= link_to "Delete", question_path(question), method: :delete, data: {confirm: "Would you like to delete this?"} %></p>
		</div>
	<% end %>
</div>

```

## Calling a partial from a partial

We could break up the 'questions' partial by introducing a 'question' partial. To do this we introduce another file app/views/questions/_question.html.erb. We can copy the HTML required to display a Question into that file:

```html
<!-- app/views/questions/_question.html.erb -->

<div>
	<h4><%= question.title %></h4>
	<p><%= format_body(question) %></p>
	<p><%= link_to(pluralize(question.answers.count, "answer"), question_answers_path(question))  %></p>
	<p><%= link_to "Show", question_path(question) %> | <%= link_to "Edit", edit_question_path(question) %> |  <%= link_to "Delete", question_path(question), method: :delete, data: {confirm: "Would you like to delete this?"} %></p>
</div>

```
The partial is called 'question' which means that we can use a variable called 'question' provided we set the ':object' parameter when rendering the partial. 

We are rendering the question partial from the questions partial:

```html
<!-- app/views/questions/_questions.html.erb -->

<div>
	<h3><%= caption %></h3> 
	<% questions.each do | question | %>  
		<%= render partial: "question", object: question %>
	<% end %>
</div>
```

According to the Rails documentation "if you have an instance of a model to render into a partial, you can use a shorthand syntax". This shorthand syntax is as follows:

```html
<!-- app/views/questions/_questions.html.erb -->

<div>
	<h3><%= caption %></h3> 
	<% questions.each do | question | %>  
		<%= render question %>
	<% end %>
</div>
```

Rails figures out the name of the partial from the type of the model, not the name of the variable.

You can do this when rendering a partial from a partial, or when rendering a partial from a view template such as index.html.erb.


## Partials and collections 

In the questions partial, we are dealing with a list of 'questions'. The render method has a `:collection` option to use when dealing with collections. 

When using this, the partial will be rendered once for each item in the collection. 

To make use of this we can update the questions partial:

```html
<!-- app/views/questions/_questions.html.erb -->

<div>
	<h3><%= caption %></h3> 
	<%= render partial: "question", collection: questions %>  
</div>
``` 

In case you were wondering, there is also a shorthand for this:


```html
<!-- app/views/questions/_questions.html.erb -->

<div>
	<h3><%= caption %></h3> 
	<%= render questions %>  
</div>
``` 

Rails determines the partial to use by looking at the type of each item (not the name of the collection variable). This means that we could use this format from within the index page if we wanted to e.g.

```html
...
...
...
<%= render @all_questions %>
...
...
...
```


As this is a complex topic, 26_partials_part_4.md provides a recap and applies this technique to displaying answers. 


