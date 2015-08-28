# Validations

Until now we have assumed that our users will inter data correctly. We have not checked that all form fields are filled in, or that certain conditions have been met e.g. someone could enter a negative number for a hotels price.

In other words, we have not validated the form inputs. Rails helps with this by providing a 'validations' system.


In Questionable, we can make the Questions 'title' field a required field.

To achieve this we begin by modifying the Question model. We can use the `validates` method from Rails:

```ruby

# app/models/question.rb
class Question < ActiveRecord::Base
	has_many :answers

	validates :title, presence: true
end
```
This will only save a Question if the title is present. We can verify this in the Rails Console.

```
$ rails c
Loading development environment (Rails 4.2.3)
2.2.2 :001 > question = Question.new
 => #<Question id: nil, title: nil, created_at: nil, updated_at: nil, body: nil> 
2.2.2 :002 > question.valid?
 => false 
2.2.2 :003 > question.save
   (0.2ms)  begin transaction
   (0.1ms)  rollback transaction
 => false 
2.2.2 :004 > question.title = "Cheese?"
 => "Cheese?" 
2.2.2 :005 > question.valid?
 => true 
2.2.2 :006 > question.save
   (0.1ms)  begin transaction
  SQL (0.3ms)  INSERT INTO "questions" ("title", "created_at", "updated_at") VALUES (?, ?, ?)  [["title", "Cheese?"], ["created_at", "2015-08-28 11:26:31.825593"], ["updated_at", "2015-08-28 11:26:31.825593"]]
   (1.4ms)  commit transaction
 => true 
```

As you can see there was a 'rollback' when we tried to save a Question which did not have a title. This means that the question was not saved. 

`question.save` returned false when the model was invalid, indicating that it did not save. `question.save` returns true when a save has been carried out successfully.

Also note the use of 'question.valid?' which is used to tell if a model is valid or not.

Each model has a list of errors which we can access as follows:

```
$ rails c
Loading development environment (Rails 4.2.3)
2.2.2 :001 > question = Question.new
 => #<Question id: nil, title: nil, created_at: nil, updated_at: nil, body: nil> 
2.2.2 :002 > question.errors.full_messages
 => [] 
2.2.2 :003 > question.valid?
 => false 
2.2.2 :004 > question.errors.full_messages
 => ["Title can't be blank"] 

```

Interestingly enough, the `errors` are empty until the validations are run. These get run when we try to make changes to the database or when `valid?` is called.

We can also get the validation errors which apply to each field:

```
2.2.2 :006 > question.errors[:title]
 => ["can't be blank"] 
2.2.2 :007 > question.errors[:body]
 => []
```

We can add another validation to ensure that the length of the 'body' is at least 10 characters.

```
# app/models/question.rb
class Question < ActiveRecord::Base
	has_many :answers

	validates :title, presence: true
	validates :body, length: { minimum: 10 }

end

```

It is worth reading the section on presence here: http://guides.rubyonrails.org/active_record_validations.html#presence

Take note, in the link above, on how to be sure that an association is present.

E.g.

```ruby
class LineItem < ActiveRecord::Base
  belongs_to :order
  validates :order, presence: true
end
```

Rails has many different validation helpers, which can be found here: http://guides.rubyonrails.org/active_record_validations.html

Here are a few more examples: 

To validate that a price is not negative use:

validates :price, numericality: { greater_than_or_equal_to: 0 }

More details can be found here: http://guides.rubyonrails.org/active_record_validations.html#numericality

The 'format' helper is one of the most powerful validation helpers. It validates based on a regular expression.

To use the format validate to validate an image name use:

```ruby
validates :image_name, allow_blank: true, format: {
	with: /\w+\.(gif|jpg|png)\z/i,
	message: "must be a valid image (GIF, JPG or PNG)"
}

```

See more information here: http://guides.rubyonrails.org/active_record_validations.html#format


## Exercise 

Add validations to your models.

Test these out in the Rails Console.