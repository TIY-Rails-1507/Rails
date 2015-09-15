# AJAX (API) Client

Next we will build a client page that can use the JSON API that is provided by 'breaking_news'. One of the difficulties to grasp is that this client page will be served by the 'breaking_news' application itself. 

In other words the server app will render this client page and serve it to the client. The page will be loaded in the browser, and then it will interact with the server through the API.

We will begin by rendering a static page and then introduce JavaScript. 

## Static Page

First we generate a controller:

```
rails generate controller static 
```
We won't need to add any actions to this controller. Rails 4 will use the default behavior and look up a view to match the action name supplied trough the routing module.


In `routes.rb`, add the following as the last route:

```ruby
get 'news' => 'static#news'
```

Now we can add a static page as a view template:

```html
<!-- app/static/news.html.erb -->
<h1>Breaking News</h1>

<p>These are the stories</p>
```


And try this URL in the browser `http://localhost:3000/news`

That should load the page and you should see the "Breaking News" header.



References: 
* http://www.peoplecancode.com/tutorials/how-to-create-static-pages-in-ruby-on-rails-application
* http://stackoverflow.com/a/4966764/259477

