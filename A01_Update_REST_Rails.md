# Details on REST and Rails


## REST
Browsers (and other web clients) use HTTP to communicate with a web server. This is why we see the 'HTTP:' part in the start of URLs. This is specifying the protocol to use. 

The full URL is what is known as an end point. You can build your end points to in any form you wish. For example if we had an application that showed information for teams we could make our web application show a list of all teams either of these URLs:
* http://example.com/all-teams
* http://example.com/teams/all
* http://example.com/show/teams
* http://example.com/global/teams
* http://example.com/12/20
* http://example.com/tms
* http://example.com/teams

All of these URLs are valid. When you design your app you need to think about which URLs you want to use. These days it is recommended that you follow a style of architecture known as REST.

Representational State Transfer (REST) was introduced by Roy Fielding to describe a standard way of creating HTTP APIs. 

When using the RESTful style we map main HTTP verbs (GET, POST, PUT, DELETE) onto common actions e.g. view, create, edit, and delete.

### HTTP methods\verbs
HTTP verbs tell the server what to do with the request being made by the client.

Commonly used verbs:
* GET
* POST
* PUT
* DELETE

Mapping onto actions:
* GET - read
* POST - create
* PUT - update
* DELETE - delete \ destroy

In REST there is the notion of a resource. In the example below, a 'team' is a resource. The resource is always in the plural form.

Examples of RESTful URLs

View all

GET http://example.com/teams

To view a single team

GET http://example.com/teams/100

Update

PUT http://example.com/teams/100


In Rails it has been decided to use PATCH instead of PUT. This means that the verb to action mapping has been changed from:

* GET - read
* POST - create
* PUT - update   <---
* DELETE - delete \ destroy

To:

* GET - read
* POST - create
* PATCH - update  <---
* DELETE - delete \ destroy

This means we use the following for conventions for an update in Rails:

PATCH http://example.com/teams/100


If you would like more details on the reasons behind the change, see these links: 
* http://weblog.rubyonrails.org/2012/2/26/edge-rails-patch-is-the-new-primary-http-method-for-updates/
* http://blog.remarkablelabs.com/2012/12/http-patch-verb-rails-4-countdown-to-2013
