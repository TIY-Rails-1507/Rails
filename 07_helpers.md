# View Helpers

Helpers perform reusable functionality which does not belong in  models, and which we don't want to duplicate in different views. 

We have already covered helpers e.g. number_to_currency. We will use another one in the session. 

You can find many more helpers and other documentation by using the search field here: http://api.rubyonrails.org/


### Adding a 'posted on' field to the Question index page.

Thanks to Rails conventions, we already have the 'created_at' field. We can use that within our HTML.

```ruby
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>


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
     <td><%= question.title%></td>
     <td><%= truncate(question.body, length: 20, separator: ' ') %></td>
     <td><%= question.created_at %></td>
  </tr>
  <% end %>  
  </tbody>
</table>
```
This shows the date in UTC format: 2015-08-15 09:30:09 UTC

While accurate, it is not that readable for the users.

We can use a Rails formatter to format the date:

```
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
     <td><%= question.title%></td>
     <td><%= truncate(question.body, length: 20, separator: ' ') %></td>
     <td><%= question.created_at.to_s(:short) %></td>
  </tr>
  <% end %>  
  </tbody>
</table>

```

### Display the number of questions in the system

The objective is to display the number of questions within the application. We want the text to read well e.g.
* 3 questions
* 1 question
* 0 questions 

Do to this we can use pluralize.

http://api.rubyonrails.org/classes/ActionView/Helpers/TextHelper.html#method-i-pluralize

```ruby
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
     <td><%= question.title%></td>
     <td><%= truncate(question.body, length: 20, separator: ' ') %></td>
     <td><%= question.created_at %></td>
  </tr>
  <% end %>  
  </tbody>
</table>
```

### Display 'n/a' if no body was supplied

Let's try to achieve this through plain ERB syntax:

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
     <td><%= question.title%></td>
     <% if question.body && question.body.length != 0  %>
	    <td><%= truncate(question.body, length: 20, separator: ' ') %></td>
	 <% else  %>
	 	<td>n/a</td>
	 <% end  %>
	 <td><%= question.created_at.to_s(:short) %></td>
  </tr>
  <% end %>  
  </tbody>
</table>

```

It would be cleaner to use a custom helper. That would read as follows:

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
     <td><%= question.title%></td>
     <td><%= format_body(question) %></td>
	 <td><%= question.created_at.to_s(:short) %></td>
  </tr>
  <% end %>  
  </tbody>
</table>
```
To create the custom helper we add the following code:

```ruby
# app/helpers/questions_helper.rb
module QuestionsHelper
	def format_body(question)
		if question.body && question.body.length != 0 
			# note you can call a helper from a helper
			truncate(question.body, length: 20, separator: ' ')
		else
			'n/a'
		end
	end
end

```
This should achieve the result we wanted.

We can shorten the code using the 'present?' method from Rails:

 ```ruby
 if question.body && question.body.length != 0 
 ```
 could be shortened to:
 ```ruby
if question.body.present? 
 ```

To check if a field is null or empty, use .blank?


Next, let's assume we wanted to put the 'n/a' in italics.

This is an attempt to embed html in the string returned form the heler:

```ruby
# app/helpers/questions_helper.rb
module QuestionsHelper
	def format_body(question)
		if question.body.present?
			# note you can call a helper from a helper
			truncate(question.body, length: 20, separator: ' ')
		else
			'<i>n/a</i>'
		end
	end
end
```
This does not work, as Rails escapes all HTML characters by default. To achieve this we can use:

```ruby
'<i>n/a</i>'.html_safe
```

Another option is to use a built in content_tag helper:

```ruby
content_tag(:i, 'n/a')
```

### Exercise 

Apply this to format the hotels price. Set the price to 'on request' if it has not been set.


