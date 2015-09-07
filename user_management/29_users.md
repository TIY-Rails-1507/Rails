# User Model

We begin by creating the Model for a user. This includes the database table to store the users.

We will store a `name` and an `email` for each user. We can generate a migration with:

```
rails generate model user name:string email:string 
```

Apply the migration with:

```
rake db:migrate
```

## Validations

### Required and length validations

We need to add validations to the user model. We need to make sure that a name and email are supplied. It is also prudent to add maximum lengths: 

```ruby
# app/models/user.rb
class User < ActiveRecord::Base
	validates :name,  presence: true, length: { maximum: 50 }
  	validates :email, presence: true, length: { maximum: 200 }
end

```
### Format validations

Next we need to make sure that the email address is in a valid format. The recommended Rails way of doing this is with a custom validation class - http://edgeguides.rubyonrails.org/active_record_validations.html#custom-validators

As explained by the Rails guides "The easiest way to add custom validators for validating individual attributes is with the convenient ActiveModel::EachValidator. In this case, the custom validator class must implement a validate_each method which takes three arguments: record, attribute, and value. These correspond to the instance, the attribute to be validated, and the value of the attribute in the passed instance."

To implement this create a new folder `app/validators` and add the following file:

```ruby
# app/validators/email_validator.rb
class EmailValidator < ActiveModel::EachValidator
  
  def validate_each(record, attribute, value)
    unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      record.errors[attribute] << (options[:message] || "is not an email")
    end
  end
  
end
```
If the value of the attribute does not match the regular expression, then an error will be added to the models validation errors.

The following is added to the model class to make use of this validator:

```ruby
# app/models/user.rb
require_dependency 'app/validators/email_validator.rb'
class User < ActiveRecord::Base
	validates :name,  presence: true, length: { maximum: 50 }
  	validates :email, presence: true, length: { maximum: 200 }
    validates :email, email: true
end

``` 

Take note of the `require_dependency` call. In this case Rails may not have auto-loaded that file, most likely because we don't instantiate it in a typical way i.e. we did not use EmailValidator.new - http://guides.rubyonrails.org/autoloading_and_reloading_constants.html#require-dependency

FYI: An interesting note on validations and and emails: http://stackoverflow.com/a/1189163/259477

### Uniqueness validations

We would like to make sure that we don't have two different registrations with the same email address. This means we need to make sure that emails are unique in the database. It is recommended to do this both at the model level and at the database level. Adding this constraint at the database level prevents erroneous errors that could result when there are concurrent database connections.

See here for more information and an example: https://robots.thoughtbot.com/the-perils-of-uniqueness-validations

To add this validation check to the model we use the following:

```ruby
# app/models/user.rb
require_dependency 'validators/email_validator.rb'
class User < ActiveRecord::Base
	validates :name,  presence: true, length: { maximum: 50 }
  	validates :email, presence: true, length: { maximum: 200 }
    validates :email, email: true
    validates :email, uniqueness: { case_sensitive: false }
end
```

To add an additional layer of protection, we can add a database constraint. This adds a validation constraint on the database which will prevent a duplicate email. In Rails, this is done through a migration:

```
$ rails generate migration AddIndexToUsersEmail
```
Or if you prefer snake case:

```
$ rails generate migration add_index_to_users_email
```

Open the new migration file and add modify it so that it has the following `add_index` line:

```ruby
class AddIndexToUsersEmail < ActiveRecord::Migration
  def change
  	add_index :users, :email, unique: true
  end
end
```

Run the migration:

```
rake db:migrate
```

While this helps it may not work for all types of database systems. This because some database engines will apply this constraint using case insensitive and other will use case sensitive matches. As a work around to this problem we will down case all emails before saving. This can be done through a special 'callback method' known as `before_save`.

```ruby
# app/models/user.rb
require_dependency 'validators/email_validator.rb'
class User < ActiveRecord::Base
	before_save :downcase_email

	validates :name,  presence: true, length: { maximum: 50 }
  	validates :email, presence: true, length: { maximum: 200 }
    validates :email, email: true
    validates :email, uniqueness: { case_sensitive: false }

    private 

    def downcase_email
    	self.email = email.downcase
    end
end
```
Rails will call `downcase_email` before creating or updating a user model.

In the code above there is this line:
```ruby
	def downcase_email
    	self.email = email.downcase
  end
```
Using `self` on the right of the equals sign is optional, however it is mandatory on the left. If we did not use `self` Ruby would think that we were defining a local variable.

Information on `before_create` and `before_save`, and the order of calls can be found here: http://stackoverflow.com/a/6249572/259477 

## Exercise 

Add user models to Explore. These users should have a 'name' and 'email' as shown above. We will add passwords in the next section.
