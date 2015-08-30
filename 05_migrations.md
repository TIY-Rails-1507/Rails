# Migrations

These allow us to modify the schema of the database e.g. adding a column, updating a column, deleting a table e.t.c.

Obviously we want to do this without deleting the data.

## Updating the Question table (and model)

To get built in help, use:

```
rails generate migration
```
This shows:
```
Usage:
  rails generate migration NAME [field[:type][:index] field[:type][:index]] [options]
```

The NAME must be something that you have not used before, or it will overwrite the previous migration file.

Using standard naming conventions for the migration name can help rails infer what we are trying to achieve. 

```
rails generate migration AddBodyToQuestion body:text
```
Text is used for text fields which require a long length - the database will use different types for body:string and body:text.

The migration file looks like this:
```
class AddBodyToQuestion < ActiveRecord::Migration
  def change
    add_column :questions, :body, :text
  end
end
```

Notice how Rails could infer the table name from the name of the migration.

Running the migration, using status to get some insights

```
rake db:migrate:status
rake db:migrate
rake db:migrate:status

```

Now we can use all the Active Records we covered before:


```
rails console
q = Question.new
q.title = "Fresh Bread?"
q.save
q.title
q.id


quest = Question.new(title: "Eggs?", body:"Where can I get good eggs?")
quest.save


question = Question.create(title: "Milk?", body: "Is milk good for me?")

question.id

Question.first
Question.count
Question.all.each { |q| puts q.title } 

question = Question.find(3)
question.body = "I am worried about milk, is it good for me?"

Question.create(title: "Red socks?", body: "Where can I find them?")
quest = Question.find_by(title: "Red socks?")
quest.update(title: "Green socks?", body: "Are green socks better than red?")
```

## Updating the view

The next thing to do is to update the view so that the body is displayed.

We can use the truncate method to help:
http://api.rubyonrails.org/classes/ActionView/Helpers/TextHelper.html#method-i-truncate

```
<!-- app/views/questions/index.html.erb -->
<h3>Questions</h3>

These are the current questions:
<ul>
<% @questions.each do |question| %>
   <li><%= question.title%> - <%= truncate(question.body, length: 20, separator: ' ') %></li>
<% end %>  
</ul>
```

## Exercise
* Add a description column to the hotel model 
* Display this in the view
