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

We will be doing this work in the context of a news site called 'Breaking News'. The main resource in the project will be a news article. let's begin by creating a new project. 

### Creating Breaking News

Make sure you are at your development root - i.e. a good location to create a new Rails project, so that you are not going to end up with nested Git folders.


To create the Rails application type:

```
rails new breaking_news
```

Start the rails server: 

```
rails s
```

Navigate to `http://localhost:3000/` and you should see the initial setup screen:

![Welcome](/images/welcome.jpg?raw=true "Welcome Screen")

Next we add this to a local git repo:

```
dev $ cd breaking_news/
breaking_news $ git init
Initialized empty Git repository in /Users/daryn/dev/breaking_news/.git/
breaking_news $ git add -A
breaking_news $ git commit -m "Initial check in"

```

From this point on you are expected to commit your work on a regular basis. If you want to commit it to GitHub please ask for help.

### Scaffolding 




