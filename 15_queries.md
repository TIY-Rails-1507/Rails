# Queries 

From this point on, you will need to start digging around for the information. This will help to improve your Rails skills as well as your searching skills. This will give you more confidence to 'teach yourself' once the course has ended. 


## Task 1

Display the hotels (or items) in descending alphabetical order on the index page - (this would be reverse alphabetically)

Ref:
* http://guides.rubyonrails.org/active_record_querying.html#ordering

## Task 2

Make the index page have two tables. One showing the 'luxury hotels' and one showing 'budget'. You can decide on a price that categorizes the hotels.

Ref:
* http://apidock.com/rails/ActiveRecord/QueryMethods/where

## Task 3

For each table, show the price range from the cheapest hotel to the most expensive

Ref:
* http://guides.rubyonrails.org/active_record_querying.html#minimum


## Task 4

It is good practice to keep the controllers 'thin' - this means to keep code out of the controllers. If you solved the tasks above by adding code to the controller, now would be a good time to refactor that. 

This logic belongs in the Hotel model. Ideally the code in the controller should look like this:

```ruby
@budget_hotels = Hotel.budget_hotels
```

That is actually a class level method, which means we would define it on the Hotel class as follows:

```ruby
 def self.budget_hotels
 	# code goes here
 	# You don't need 'Hotel', only the where
 	# This is class method, 'self' is the class Hotel 
 end

```

Move as much code as possible out of the controller into the model.


