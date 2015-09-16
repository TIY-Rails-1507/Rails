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

<div id="articles-list"></div>
```

And try this URL in the browser `http://localhost:3000/news`.  That should load the page and you should see the "Breaking News" header.


## HTTP Request with JQuery

Next we need to add the JavaScript needed to request information from the server. We can begin by adding a JavaScript file to `app/assets/javascripts` called `news.js`.

```js
// app/assets/javascripts/news.js

alert("Loaded");

```

This is an initial test to make sure that the JavaScript file is being included. Refreshing the page should cause this alert to show.

Next we need to make a request to the server to fetch the data. This can be done in plain JavaScript, however JQuery makes this easier. 


```js
// app/assets/javascripts/news.js

$(document).on('ready page:change', function () {
     type: "GET",
     contentType: "application/json; charset=utf-8",
     url: "/news.json",
     dataType: "json",
     success: function (result) {
        //do somthing here
        window.alert("success!!");
     },
     error: function (){
        window.alert("something wrong!");
     }
});

```



References: 
* http://www.peoplecancode.com/tutorials/how-to-create-static-pages-in-ruby-on-rails-application
* http://stackoverflow.com/a/4966764/259477
* http://railscasts.com/episodes/279-understanding-the-asset-

Turbolinks  
* http://guides.rubyonrails.org/working_with_javascript_in_rails.html#turbolinks

**Using Page Load** - Read the 'edit' section on moving delegated events outside of page load
* http://stackoverflow.com/a/19834224/259477

Alternative way of 'page load'  
* http://stackoverflow.com/a/18834209/259477

Making a post with JQuery  
* http://stackoverflow.com/questions/20087584/jquery-and-rails-ajax-request-and-response

Stringify 
* https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify