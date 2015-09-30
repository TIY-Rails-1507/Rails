# Using remote

In this section we use a second approach to performing AJAX calls within Rails. We be using JSON directly we will be using some built in AJAX support.


## Creating a partial

Before we can start it would be useful to create a partial. The articles index page currently shows a table. Each row represents an article. We will extract the row into a partial. Add the following to a new file:

```ruby
<!-- app/views/articles/_article.html.erb -->

<tr>
	<td><%= article.headline %></td>
	<td><%= article.body %></td>
	<td><%= link_to 'Show', article %></td>
	<td><%= link_to 'Edit', edit_article_path(article) %></td>
	<td><%= link_to 'Destroy', article, method: :delete, data: { confirm: 'Are you sure?' } %></td>
</tr>
``` 

Now we can update the index to page so that it has the following:

```html
<p id="notice"><%= notice %></p>

<h1>Listing Articles</h1>

<table>
  <thead>
    <tr>
      <th>Headline</th>
      <th>Body</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <%= render @articles %>
  </tbody>
</table>

<br>

<%= link_to 'New Article', new_article_path %>

```

Verify that the index page still displays correctly.

## Submitting a form with AJAX

Change the `new` view so that it has the following:

```html
<h1>New Article</h1>

<%= render 'form' %>

<%= link_to 'Back', articles_path %>


<h1>Listing Articles</h1>

<table>
  <thead>
    <tr>
      <th>Headline</th>
      <th>Body</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <%= render @articles %>
  </tbody>
</table>

<br>

<%= link_to 'New Article', new_article_path %>
```

Because this is using a variable called `@articles` we need to set that in the `new` action in the controller controller:

```ruby
  # GET /articles/new
  def new
    @article = Article.new
    @articles = Article.all
  end
```

As the 'new' form now shows all articles, we could redirect to this view after creating one. To do this change the 'create` action:

```ruby
  # POST /articles
  # POST /articles.json
  def create
    @article = Article.new(article_params)

    respond_to do |format|
      if @article.save
        format.html { redirect_to new_article_path, notice: 'Article was successfully created.' }
        format.json { render :show, status: :created, location: @article }
      else
        format.html { render :new }
        format.json { render json: @article.errors, status: :unprocessable_entity }
      end
    end
  end

```

While this gives the appearance of an AJAX because Rails uses a gem called Turbolinks which helps to speed up page loads. To make an AJAX call we need to make further changes. 

## Submitting a form with AJAX

Open the `_form` partial and change the first line from:

```html
<%= form_for(@article) do |f| %>
``` 
To: 

```html
<%= form_for(@article, remote: true) do |f| %>
``` 

This will change the HTML that is rendered from this:

```html
<form class="new_article" id="new_article" action="/articles" accept-charset="UTF-8" method="post">
```

To:

```html
<form class="new_article" id="new_article" action="/articles" accept-charset="UTF-8" data-remote="true" method="post">
```

The `data-remote="true"` attribute changes the way this form is submitted. 'Rails & JQuery' will now submit this form through an AJAX request and not the standard request. 

This will now use a 'js' mime type which means we need add code to the controller to respond to a 'js' request. Add `format.js` to the `create` action in the controller.

```ruby
  # POST /articles
  # POST /articles.json
  def create
    @article = Article.new(article_params)

    respond_to do |format|
      if @article.save
        format.html { redirect_to @article, notice: 'Article was successfully created.' }
        format.json { render :show, status: :created, location: @article }
        format.js   {} 
      else
        format.html { render :new }
        format.json { render json: @article.errors, status: :unprocessable_entity }
      end
    end
  end
```

Using `format.js` will result in Rails returning a matching `*.js.erb` file. For the create action it will return `app/views/articles/create.js.erb`. Make that file and add the following code:

```js
// app/views/articles/create.js.erb
alert("This file is generated on the server but executed on the client.");
```

Just as a view is rendered in the browser, this code is run in the browser following a successful creation. 

What we would like to do is append HTML to the table on the `new` page. We can use JQuery to append HTML to the table by using `$('tbody').append(...);`

That will find the `tbody` element and add what ever HTML it is provided with. This HTML will be added just before the closing tbody tag.

This means we need to give `append` HTML which matches a table row. That would look something like this: 

```html
<tr>
	<td>Ruby</td>
	<td>Ruby is a great programming language</td>
	<td><a href="/articles/1">Show</a></td>
	<td><a href="/articles/1/edit">Edit</a></td>
	<td><a data-confirm="Are you sure?" rel="nofollow" data-method="delete" href="/articles/1">Destroy</a></td>
</tr>
```

You may recall that this HTML matches what the `_article` partial generates. We can use that in this script by doing the following:

```js
// app/views/articles/create.js.erb
$('tbody').append("<%= j(render @article) %>");
```

That will render the `_article` template which produces the HTML that we need. The `j` is a method call that will escape any characters that could make the response result in invalid JavaScript. The `j` is actually a synonym for `escape_javascript`.

# Exercise 

Use `remote: true` on an update link. When that is clicked a 'form' should appear that will let a user edit the article. 

