# Passwords for Users

When a user signs in they need to provide a password. This authenticates the user i.e. verifies that the user is who they claim to be.

It is poor practice to store the plain text version of users passwords in a database. If a 'hacker' gets access to the database they will see all the users passwords. It is prudent to save an encrypted version of the password in the database.

This is typically done by using a cryptographic hash - completely different to a Hash structure in Ruby. A cryptographic hash is a way of encrypting a plain text message (word or phrase). A hashed message is often known as a message digest or password digest.

Unlike some other encryption techniques a password digest can't be decrypted. This means that you cannot work out the original plain text even if you have the encrypted version. 

This 'one way' hash offers good protection for the users passwords. The downside is that not even the site knows the actual password, only the encrypted version. Therefore to sign a user in the site needs to hash the provided password again and compare that to the password digest in the database.

## Creating a hash

Rails assists with creating this functionality. It does this with a method called `has_secure_password`. This is added to the model:

```ruby
# app/models/user.rb
require_dependency 'validators/email_validator.rb'
class User < ActiveRecord::Base
	before_save :downcase_email

	validates :name,  presence: true, length: { maximum: 50 }
  	validates :email, presence: true, length: { maximum: 200 }
    validates :email, email: true
    validates :email, uniqueness: { case_sensitive: false }

    has_secure_password

    private 

    def downcase_email
    	self.email = email.downcase
    end
end

```
This adds the following pair of attributes to the model:
* password
* password_confitmation

These fields are known as 'virtual'. In this context virtual means that they exist in the model but not in the database table.

These fields have the following validations applied to them:
* required
* length must be less than or equal to 72 characters
* password must match password_confirmation 

These validations can be configured, see the documentation referenced below.

When we include `password_confitmation` then Rails will generate a method called `authenticate` which can verify that the correct password was supplied for a given user.

We need to make a few more changes before all of this is available. `has_secure_password` requires the model to have a field in the database table called `password_digest`. This will store the hashed version of the password. 

We can add this with a standard migration:

```
$ rails generate migration AddPasswordDigestToUsers password_digest:string

```

The new migration looks like this:

```ruby
class AddPasswordDigestToUsers < ActiveRecord::Migration
  def change
    add_column :users, :password_digest, :string
  end
end
```
Apply the migration with:

```
rake db:migrate
```

There are a few different algorithms for creating a hash - Rails uses bcrypt. This functionality is provided via a gem. Search your Gemfile for 'bcrypt' and uncomment that line:

```ruby
# Use ActiveModel has_secure_password
gem 'bcrypt', '~> 3.1.7'

```

Be sure to run `bundle install` to install the required bcrypt gem.


## Adding users

Now we can add some users through the Rails console:

```
questionable $ rails c
Loading development environment (Rails 4.2.3)
2.2.2 :001 > user = User.new
 => #<User id: nil, name: nil, email: nil, created_at: nil, updated_at: nil, password_digest: nil> 
2.2.2 :002 > user.name = "Sue"
 => "Sue" 
2.2.2 :003 > user.email = "SUE@YAHOO.com"
 => "SUE@GMAIL.com" 
2.2.2 :004 > user.password = "MyPassword"
 => "MyPassword" 
2.2.2 :005 > user.password_confirmation = "MyPassword"
 => "MyPassword" 
2.2.2 :006 > user.save
"SUE@YAHOO.com"
 => "SUE@YAHOO.com" 
2.2.2 :009 > user.save
   (0.1ms)  begin transaction
  User Exists (0.3ms)  SELECT  1 AS one FROM "users" WHERE LOWER("users"."email") = LOWER('SUE@YAHOO.com') LIMIT 1
  SQL (0.5ms)  INSERT INTO "users" ("name", "email", "password_digest", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?)  [["name", "Sue"], ["email", "sue@yahoo.com"], ["password_digest", "$2a$10$EZFIozMxbqCUL8NDCHQB9uzIA.uyQhnCmBuNbzOrLKgjxCJrSuEGi"], ["created_at", "2015-09-06 13:07:28.735631"], ["updated_at", "2015-09-06 13:07:28.735631"]]
   (1.5ms)  commit transaction
 => true 
```
Take note that the password is not saved, however there is a cryptographic hash saved in the `password_digest` field.

The following examples focus on the 'password confirmation':

```
2.2.2 :022 > usr = User.create(name: "clair", email: "clair@hotmail.com", password: "dontshow", password_confirmation: "incorrect")
   (0.1ms)  begin transaction
  User Exists (0.1ms)  SELECT  1 AS one FROM "users" WHERE LOWER("users"."email") = LOWER('clair@hotmail.com') LIMIT 1
   (0.1ms)  rollback transaction
 => #<User id: nil, name: "clair", email: "clair@hotmail.com", created_at: nil, updated_at: nil, password_digest: "$2a$10$I3v3qx12ijlWYz.Aii7w5eMCv4KKQggRB3P0z1S4cpk..."> 
2.2.2 :023 > usr.errors[:password_confirmation]
 => ["doesn't match Password"] 

```
This user was not saved as the password confirmation was incorrect.

```
2.2.2 :019 > User.create(name: "mark", email: "mark@hotmail.com", password: "dontshow")
   (0.1ms)  begin transaction
  User Exists (0.2ms)  SELECT  1 AS one FROM "users" WHERE LOWER("users"."email") = LOWER('mark@hotmail.com') LIMIT 1
  SQL (0.3ms)  INSERT INTO "users" ("name", "email", "password_digest", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?)  [["name", "mark"], ["email", "mark@hotmail.com"], ["password_digest", "$2a$10$c26j.meZmhtSoS65014ZlOlFpaNISH.MPNEYEiuFUC2M4xrHK9NUq"], ["created_at", "2015-09-06 13:10:52.337099"], ["updated_at", "2015-09-06 13:10:52.337099"]]
   (0.7ms)  commit transaction
 => #<User id: 5, name: "mark", email: "mark@hotmail.com", created_at: "2015-09-06 13:10:52", updated_at: "2015-09-06 13:10:52", password_digest: "$2a$10$c26j.meZmhtSoS65014ZlOlFpaNISH.MPNEYEiuFUC2..."> 

```

This user was saved. No password confirmation was supplied, therefore Rails ignored that check.

"Rails 4 completely ignores the confirmation if you don't send it as a parameter so if you forget to whitelist the password_confirmation attribute, it will be ignored and your validations won't say a thing. Careful of this!" - This is a comment from Stack Overflow - http://stackoverflow.com/a/18863258/259477

The white listing mentioned here is the `params.require ...` line which is used in the controllers.


## Authentication

Now that we have a few users in the database we can test the `authenticate` method which Rails provides when using `has_secure_password`. 

The `authenticate` method is a way to determine if the password is valid for a given user. This method will first hash the password and then compare the hash to the stored password_digest.

```
$ rails c
Loading development environment (Rails 4.2.3)
2.2.2 :001 > User.create(name: "Mike", email: "mike@hotmail.com", password: "mike_pwd", password_confirmation: "mike_pwd")
   (0.1ms)  begin transaction
  User Exists (0.2ms)  SELECT  1 AS one FROM "users" WHERE LOWER("users"."email") = LOWER('mike@hotmail.com') LIMIT 1
  SQL (0.4ms)  INSERT INTO "users" ("name", "email", "password_digest", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?)  [["name", "Mike"], ["email", "mike@hotmail.com"], ["password_digest", "$2a$10$QunZ8UGqohZSPryekV3FPuAww1eu6N8/84UL1u/YwIp0DjKT/QbAy"], ["created_at", "2015-09-06 13:35:54.794011"], ["updated_at", "2015-09-06 13:35:54.794011"]]
   (0.5ms)  commit transaction
 => #<User id: 6, name: "Mike", email: "mike@hotmail.com", created_at: "2015-09-06 13:35:54", updated_at: "2015-09-06 13:35:54", password_digest: "$2a$10$QunZ8UGqohZSPryekV3FPuAww1eu6N8/84UL1u/YwIp..."> 
 
 
 2.2.2 :003 > mike = User.find_by(name: "Mike")
  User Load (0.1ms)  SELECT  "users".* FROM "users" WHERE "users"."name" = ? LIMIT 1  [["name", "Mike"]]
 => #<User id: 6, name: "Mike", email: "mike@hotmail.com", created_at: "2015-09-06 13:35:54", updated_at: "2015-09-06 13:35:54", password_digest: "$2a$10$QunZ8UGqohZSPryekV3FPuAww1eu6N8/84UL1u/YwIp..."> 

2.2.2 :004 > mike.authenticate("invalid")
 => false 
2.2.2 :005 > mike.authenticate("MIKE_PWD")
 => false 
2.2.2 :006 > mike.authenticate("mike_pwd")
 => #<User id: 6, name: "Mike", email: "mike@hotmail.com", created_at: "2015-09-06 13:35:54", updated_at: "2015-09-06 13:35:54", password_digest: "$2a$10$QunZ8UGqohZSPryekV3FPuAww1eu6N8/84UL1u/YwIp..."> 


2.2.2 :007 > sue = User.find_by(email: "sue@yahoo.com")
  User Load (0.2ms)  SELECT  "users".* FROM "users" WHERE "users"."email" = ? LIMIT 1  [["email", "sue@yahoo.com"]]
 => #<User id: 4, name: "Sue", email: "sue@yahoo.com", created_at: "2015-09-06 13:07:28", updated_at: "2015-09-06 13:07:28", password_digest: "$2a$10$EZFIozMxbqCUL8NDCHQB9uzIA.uyQhnCmBuNbzOrLKg..."> 

2.2.2 :008 > sue.authenticate("what")
 => false 
2.2.2 :009 > sue.authenticate("mypassword")
 => false 
2.2.2 :010 > sue.authenticate("MyPassword")
 => #<User id: 4, name: "Sue", email: "sue@yahoo.com", created_at: "2015-09-06 13:07:28", updated_at: "2015-09-06 13:07:28", password_digest: "$2a$10$EZFIozMxbqCUL8NDCHQB9uzIA.uyQhnCmBuNbzOrLKg..."> 
```


Note: If the correct password is given then the user object is returned, when an invalid password is given false is returned. This can be used in the controller when signing in a user.




#### Notes

If you want to ensure a minimum password length you can add the following validation to the user Model:

```
validates :password, length: { minimum: 5 }
```

`has_secure_password` documentation: http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html

bcrypt documentation: https://en.wikipedia.org/wiki/Bcrypt


## Exercise

Add `has_secure_password` to the User model in Explore and apply the changes mentioned above. Add a few valid users - take a note of the password and keep them simple.

