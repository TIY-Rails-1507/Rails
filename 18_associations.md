# Associations 


It is common to have items which are related to other items. For example a team has players, a question has answers and a hotel has reviews.


The Questions table looks similar to the following diagram:

![Questions table](/table_images/questions.png?raw=true)

We would like to create an Answers table. Each answer refers to a question. This is done by adding a 'question_id' to the Answers table:

![Questions and Answers table](/table_images/highlight.png?raw=true)

In 'rails speak' we would say that a Question has many Answers. An Answer belongs to one Question.

In traditional SQL (database) terms, we would say that this is a one to many relationship i.e. one question has many answers. 

To be precise one question can have zero, one or many answers.

The answer table has an 'id' column. This column is called the primary key.

Within the answer table, the 'answer_id' column is known as a foreign key column. It has entries which are considered as foreign keys e.g. primary keys from another table.   

To make sure you understand this, try and figure out which Answers belong to which Questions in the previous images.


## Generating the model and table

We will begin by generating the 'Answer' model and the 'Answers' table.

We are going to give the Answer a property called 'body'. It is a pure coincidence that a Question also has a 'body' - please choose which ever column name you feel fits your project best.

We will also add the reference to the Questions table, this is done with the following line in the console: 

```
rails generate model answer body:text question:references
```

Note the end of the line: `question:references`. Read this as 'the answer table references the question table'. 

This created a migration file and a model file.

The migration file can be found in app/migrate/<time_stamp>_create_answers.rb

It looks as follows:

```ruby

class CreateAnswers < ActiveRecord::Migration
  def change
    create_table :answers do |t|
      t.text :body
      t.references :question, index: true, foreign_key: true

      t.timestamps null: false
    end
  end
end
```
In this migration file, we can see a line which will generate the 'foreign key' column within the table. Even though the migration file has this labeled as ':question' this column will be called 'question_id' in the database table.

Next we can apply the migration:

```
rake db:migrate
```

You may recall that you can see the status of the migrations using:
```
rake db:migrate:status
```

Next, I will examine the model in the rails console:

```
$ rails c
Loading development environment (Rails 4.2.3)
2.2.2 :001 > Answer
 => Answer(id: integer, body: text, question_id: integer, created_at: datetime, updated_at: datetime) 

```

The question_id column is there as expected.

## Setting up the models

The Answer model file looks interesting:

```ruby
# app/models/answer.rb
class Answer < ActiveRecord::Base
  belongs_to :question
end
```

When we generated this file, Rails added a 'belongs_to' entry. This tells the model that has a reference to another model. 

See more information here: http://guides.rubyonrails.org/association_basics.html#the-belongs-to-association


The Question model does not have a reference to answers:

```ruby
# app/models/question.rb
class Question < ActiveRecord::Base
end
```

To fix this we can use the 'has_many' association

http://guides.rubyonrails.org/association_basics.html#the-has-many-association

This would change the code to the following:

```ruby
# app/models/question.rb
class Question < ActiveRecord::Base
	has_many :answers
end
```

The 'belongs_to' and 'has_many' association lines add additional methods and behaviors to the models. We will see these in action in the next section.


If we delete a question, we would want all related answers to also be deleted. To get this behavior we use additional options on the has_many method:


```ruby
# app/models/question.rb
class Question < ActiveRecord::Base
	has_many :answers, dependent: :destroy
end
```

Deleting a question deletes all questions, but we don't want deleting an answer to delete the question.  


## Experimenting with the associations

As I already have an existing rails console session open, I either need to exit it or reload it by typing 'reload!'

```
2.2.2 :002 > reload!
Reloading...
 => true 
2.2.2 :003 >
```

Now I can test these associations withing the rails console

```
answer = Answer.new
answer.body = "Yes, milk is good in moderation"

question = Question.find(3)

question.answers << answer
question.save

question.answers[0].body
question.answers.first.body

question.answers[0].question_id

ans = question.answers.build(body: "I am not sure")
ans.question_id
question.save

a = Answer.find(1)
a.question.title


question.answers.create(body: "You should ask a doctor")

question.reload

question.destroy

```

## Exercise 

Add reviews to the hotels. Do this at the model level for now, we will update the views in the next section.







