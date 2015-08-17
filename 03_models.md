# Models

These notes are related to 'Questionable', you need to apply the concepts to 'Explore'.

We will use active record which is an Object Relation Mapper (ORM). This gives us an object oriented way of interacting with the database. In other words, we don't need to write SQL - well, only on special occasions...

## Generate a model

In the Questionable app, we would like to store 'questions'. This is the first model in the app.

Let's assume the question model has one field, 'title'.

This next line generates the question model:

```ruby
rails generate model question title:string
```
Note: This name is singular 

This generated a number of files, notably
* app/models/question.rb
  * This is the actual model
* db/migrate/'time stamp'_create_questions.rb
  * This file is used to create the table in the database

The model file is empty:
```ruby
# app/models/question.rb
class Question < ActiveRecord::Base
end
```
Rails 'adds' methods to this class at run time.

Looking at the migration file we have:

```ruby
# db/migrate/'time stamp'_create_questions.rb
class CreateQuestions < ActiveRecord::Migration
  def change
    create_table :questions do |t|
      t.string :title

      t.timestamps null: false
    end
  end
end
```
This file can create a new table with the supplied fields.

Every row in a table should have a unique ID, and Rails will add one for us by default.

```t.timestamps null: false```
This gets added by default. When the table is created it will add entries for 'created_at' and 'updated_at'. Rails takes care of creating and updating the data in these fields.


This file can now be run against most relational database management systems to create a table.

Use the following commands to see the migration status, and to apply the migration:

```
rake db:migrate:status
rake db:migrate
rake db:migrate:status

```
Rails keeps track of the applied migrations in a schema migrations table.

By default rails uses sqlite3. This is a no mess not fuss, file based database. It is the quickest to get started with.

To see the configuration go into: 

```
app/config/database.yml

```
In this file we can see how to use different databases in different environments e.g. development, test and production. We will make these changes at a later stage.


## Put the model to work
Using the rails console:

```
rails console
q = Question.new
q.title = "Fresh Bread?"
q.save
q.title
q.id



quest = Question.new(title: "Eggs?")
quest.save


question = Question.create(title: "Milk?")

question.id

Question.first
Question.count
Question.all.each { |q| puts q.title } 

question = Question.find(3)
question.title = "Worried about Milk"


Question.create(title: "Red socks?")
quest = Question.find_by(title: "Red socks?")
quest.update(title: "Green socks?")
```


## Exercise

* Add an 'hotel' model to the Explore application
* Run the migration file to create the table
* Use the Rails console to create hotels

Hints: 
* Use 'decimal' for a price field
* If you can't see the value of a price in the console, use the "to_s" function e.g. hotel.price.to_s





