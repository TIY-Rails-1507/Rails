# Partials - part 1

In this session we create a 'partial' which will let us 'share' code in the views. This allows us to reduce code duplication.

Partial files are ERB files and there file names start with an underscore. 

Next, we create a file in app/views/questions and call it `_form.html.erb`

We can copy the form code from edit.html.erb, into this file:

```html
<!-- app/views/questions/_form.html.erb -->

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

Next we can 'render' this partial within the edit.html.erb file:

```html
<!-- app/views/questions/edit.html.erb -->
<h3><%= @question.title %></h3>

<%= render('form') %>
```

Note that there is no underscore here. 

This method will then pull in and render the referenced partial.

We can make a similar change to new.html.erb

```html
<!-- app/views/questions/edit.html.erb -->
<h3>New Question</h3>

<%= render('form') %>

```

We can see the benefits of this by making some changes to the partial. We can use `autofocus` to make sure that the title field start off highlighted:

```
<!-- app/views/questions/_form.html.erb -->

<%= form_for(@question) do |f| %>
	<p>
		<%= f.label :title %>
		<%= f.text_field :title, autofocus: true %>
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

This has now effected the `new` and `edit` pages.

## Exercise 

Use a partial to clean up your code in Explore.


Try adding a link back to the index page to let users 'cancel'.

