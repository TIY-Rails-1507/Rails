# Polling for Changes

At the moment our users would need to refresh the page in order to get the latest news articles. It would be great if the page simply kept itself up to date with the latest articles - then it will deserve the name 'Breaking News'.

To achieve this the page needs to periodically check for updates. This is known as 'polling'. The page will periodically poll the server for new data.


## Polling the server

To poll we will be using the `setTimeout` method from JavaScript. This waits for a given amount of milliseconds and then executes the provided JavaScript. 

The first thing we would like to do is to extract the loading functionality into a new function. That would change the code from this:

```js
// app/assets/javascripts/news.js

$(document).on('ready page:ready', function () {
	if($("#articles-list").length > 0) {
		$.ajax({
			type: "GET",
			contentType: "application/json; charset=utf-8",
			url: "/articles.json",
			dataType: "json",
			success: function (result) {

				var innerHtml = "";
				for(var i = 0; i < result.length; i++) {
					innerHtml += "<div>";
					innerHtml += "<div><h4>" + result[i].headline + "</h4></div>";
					innerHtml += "<div>" + result[i].body + "</div>";
					innerHtml += "</div>";
					innerHtml += "<hr />";
				}
				$( "#articles-list" ).html(innerHtml);
			},
			error: function (){
				window.alert("There was an error!");
			}
		});
	} 
});
```

To:

```js
// app/assets/javascripts/news.js

$(document).on('ready page:ready', function () {
	if($("#articles-list").length > 0) {
		fetchArticles();
	} 
});

function fetchArticles() {
	$.ajax({
		type: "GET",
		contentType: "application/json; charset=utf-8",
		url: "/articles.json",
		dataType: "json",
		success: function (result) {

			var innerHtml = "";
			for(var i = 0; i < result.length; i++) {
				innerHtml += "<div>";
				innerHtml += "<div><h4>" + result[i].headline + "</h4></div>";
				innerHtml += "<div>" + result[i].body + "</div>";
				innerHtml += "</div>";
				innerHtml += "<hr />";
			}
			$( "#articles-list" ).html(innerHtml);
		},
		error: function (){
			window.alert("There was an error!");
		}
	});
}

```

We will use the `setTimeout` function within the `fetchArticles` function like this:

```js
// app/assets/javascripts/news.js

$(document).on('ready page:ready', function () {
	if($("#articles-list").length > 0) {
		fetchArticles();
	} 
});

function fetchArticles() {
	$.ajax({
		type: "GET",
		contentType: "application/json; charset=utf-8",
		url: "/articles.json",
		dataType: "json",
		success: function (result) {

			var innerHtml = "";
			for(var i = 0; i < result.length; i++) {
				innerHtml += "<div>";
				innerHtml += "<div><h4>" + result[i].headline + "</h4></div>";
				innerHtml += "<div>" + result[i].body + "</div>";
				innerHtml += "</div>";
				innerHtml += "<hr />";
			}
			$( "#articles-list" ).html(innerHtml);

			setTimeout(fetchArticles,5000); // A recursive function call
		},
		error: function (){
			window.alert("There was an error!");
		}
	});
}

```

At the end of the success function `setTimeout` is used. This will wait 5000 milliseconds (5 seconds) and then call `fetchArticles` again. This means that `fetchArticles` is actually calling itself. This is known as a recursive call. This is not a true recursive call because of how `setTimeout` - see the 'Memory leak?' reference below.   

You can test this by leaving the 'news' window open and adding or updating an article in another tab \ window.

### Homework Challenge - i.e. this is for homework

1. Apply the 'yellow fade' technique to any new article.

2. Apply the 'yellow fade' technique to any article that has been updated.

Hint: Ask Daryn for hints <-- almost a recursive hint :)


#### References

Discussion on polling with `setInterval` - http://stackoverflow.com/a/6835879/259477  
That same page shows different ways of polling.

Memory leak? - http://stackoverflow.com/a/12209319/259477