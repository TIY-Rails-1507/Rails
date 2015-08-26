# Queries 


## Task 1

Display the hotels (or items) in descending alphabetical order on the index page - (this would be reverse alphabetically)

Ref:
* http://guides.rubyonrails.org/active_record_querying.html#ordering

## Task 2

Make the index page have two tables. One showing the 'luxury hotels' and one showing 'budget hotels'. You can decide on a price that differentiates the hotels.

Ref:
* http://apidock.com/rails/ActiveRecord/QueryMethods/where

## Task 3

For each table, show the price range above the table. This would be from the cheapest hotel to the most expensive.

Ref:
* http://guides.rubyonrails.org/active_record_querying.html#minimum


## Task 4

It is good practice to keep the controllers 'thin' - in other words we don't want a lot of code to build up within the controllers as this becomes difficult to maintain. 

If you solved the tasks above by adding code to the controller, now would be a good time to refactor that. 

This logic belongs in the Hotel model. Ideally the code in the controller should look like this:

```ruby
@budget_hotels = Hotel.budget_hotels
```
Looking at that, you may notice that Hotel is a class and yet we are expecting to call a method on it. In Ruby it is possible to define a class level method. To define this method on the Hotel class we do the following:

```ruby
 def self.budget_hotels
 	# code goes here
 	# You don't need 'Hotel', only the where
 	# This is class method, 'self' refers to the class e.g. Hotel 
 end

```

Move as much code as possible out of the controller into the Hotel model.


