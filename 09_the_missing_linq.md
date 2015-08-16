## Linking the pages

The objective of this section is to link the pages together.

To link from the details view back to the index list, we can use the rails helper link_to - http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to

This is a method that generates an HTML anchor tag - also known as a link.

```
<!-- app/views/questions/show.html.erb -->

<h3><%= @question.title %></h3>

<p><%= @question.body %></p>

<%= link_to "Back to Questions", "/questions" %>
```

It is not ideal to had code "/questions". A more maintainable way would be to make better use of the built in routing system. The routing system can generate a URL or Path based on a route name.  

To see the routes in more detail brows to: http://localhost:3000/rails/info/routes

On that URL we can see a helper method. We can use that to generate the path as follows:

```
<!-- app/views/questions/show.html.erb -->

<h3><%= @question.title %></h3>

<p><%= @question.body %></p>

<%= link_to "Back to Questions", questions_path %>
```

This approach is more maintainable. If the path to the index page changes, we can simply update our routes.rb file and the links will automatically update.  

You can experiment with this in the rails console.

Try:
```
app.questions_url
app.questions_path
```

Next we can add links to the index page which will link all questions to there respective details page.

Looking at: http://localhost:3000/rails/info/routes

We can see that there is no helper method for 'questions/:id'. That is because rails was not able to assign a default name to this route. We can explicitly assign a name to the route by doing the following:

```ruby
# app/config/routes.rb
get 'questions' => 'questions#index'
get 'questions/:id' => 'questions#show', as: question
``` 

To confirm that the new helper method is available, go back to: http://localhost:3000/rails/info/routes

This can be used in the index page:

```html
<!-- app/views/questions/index.html.erb -->
<h3><%= pluralize(@questions.count, 'Question') %></h3>
<table border="1" >
  <thead>
  <tr>
     <th>Title</th>
     <th>Body</th>
     <th>Posted On</th>
  </tr>
  </thead>
  <tbody>
  <% @questions.each do |question| %>
  <tr>
     <td><%= link_to(question.title,  question_path(question.id)) %></td>
     <td><%= format_body(question) %></td>
	   <td><%= question.created_at.to_s(:short) %></td>
  </tr>
  <% end %>  
  </tbody>
</table>
```
If the question_path method is passed a model, it will call 'id' on the model. This means we can change:

```html
<%= link_to(question.title,  question_path(question.id)) %>
```

to:

```html
<%= link_to(question.title,  question_path(question)) %>
```

We can make one last, possibly surprising, optimization. If the second parameter to link_to is a model, then Rails assumes that we want to get the path of question - and will call 'question_path'. 

This means we can change:
```html
<%= link_to(question.title,  question_path(question)) %>
```
to:
```html
<%= link_to(question.title,  question) %>
```

## Exercise

* Link up the hotels details page to the index page
* Add links from the hotels details page to get to the details page
* Add a root route which maps onto the hotels index page
