# AJAX

AJAX stands for Asynchronous JavaScript and XML. AJAX is a term used to describe a method of updating the contents of a web page without reloading the entire page. While AJAX has XML in the name, this method is not restricted to that format. Modern web applications use a format known as JavaScript Object Notation (JSON). 

The following diagrams give a visual representation of traditional web requests and AJAX calls.

![Traditional](http://www.xml.com/2005/10/05/graphics/traditionalmodel.gif)

![AJAX](http://www.xml.com/2005/10/05/graphics/ajaxmodel.gif)

Source: http://www.xml.com/lpt/a/1622

We will be looking at two different ways of making AJAX calls within Rails. Firstly we will fetch JSON formatted data from the server and use that to update the page. 

The second method we will use is fetching a snippet of HTML from the server and then rendering that on the page. 

In both scenarios the page will get updated without us needing to refresh the page. We will be doing this work in a new project called 'Breaking News'.

## Breaking News


