# Testing in Rails

This section covers the fundamentals of testing. Automatically testing an application gives us a high level of confidence that we have not introduced defects. 

Testing that previous functionality has not been effected is known as regression testing. This can take days, or even weeks, for teams that do not have automated tests. 

There is a development technique known as TDD - this used to stand for Test Driven Development and was later adapted to Test Driven Design. Writing tests first can effect the way we design our code. It encourages encapsulation and modularized code. TDD was first popularized by Kent Beck in Extreme Programming Explained, and later in TDD by example. 

TDD popularized the practice of writing tests which run extremely fast against a unit of the application. These are known as unit tests. Unfortunately there are many conflicting opinions as to what a "unit" is. Some say it is one class and rely heavily on mocking. Others are less strict and only use mocking if the tests are slow, or there is no alternative - personally I tend to agree with the later. 

Technically it is better to test an application from the user interface layer, and work your way down. This is known as 'outside-in testing'. This approach is promoted by a development technique known as Behavior Driven Development (BDD). 


BDD was originally created to help teams adopt TDD. Some people say that BDD is TDD done properly, some people say that it is more than that. Feel free to judge for yourself - http://dannorth.net/introducing-bdd/. 

_*DD?_   

Not only is there TDD and BDD, there is also DDD (Domain Driven Design), Feature Driven Design (FDD), and a few others. Luckily all of these approaches support the same fundamental concepts - code from the outside in, from the perspective of the user.   

As this is our first exposure to testing in Rails it may be easier to grasp the concepts starting from the model and working our way up the layers.

## Setup

In this section we will start with a new application called RWC - short for Rugby World Cup. RWC will list the teams in the tournament. 

Each team will have a ranking. The ranking is a percent where the higher the number the better The application will try to predict a winner of a game, based on the raking.

Reference: http://www.worldrugby.org/rankings

### Creating the application 

Run the following command to create the application:

```
$ rails new rwc
```

Move into the new applications directory:
```
$ cd rwc
```

You may have noticed that Rails has generated a number of test folders e.g.

```
create  test/fixtures
create  test/fixtures/.keep
create  test/controllers
create  test/controllers/.keep
create  test/mailers
create  test/mailers/.keep
create  test/models
create  test/models/.keep
create  test/helpers
create  test/helpers/.keep
create  test/integration
create  test/integration/.keep
create  test/test_helper.rb

```

We will use a number of these folders later on. Another interesting observation is in `config/database.yml`. That file specifies database settings for three different environments namely development, test, and production. 

Until now we have been using the development settings on our machines. We don't actually make use of production as we deploy to Heroku, if we were deploying to another hosting service then the production environment may come into use. In the section we will be creating running tests which will run using the test settings. In short, this gives us three separate databases to use in different contexts.


## Creating the model

While we could use `generate scaffold` we will take smaller steps and generate the layers as we need them. Type the following in the terminal:

```
$ rails generate model team name:string rank:decimal
```
Output

```
invoke  active_record
create    db/migrate/20151004143516_create_teams.rb
create    app/models/team.rb
invoke    test_unit
create      test/models/team_test.rb
create      test/fixtures/teams.yml
```

Not only was the new model and migration created, but some test files were added as well. Opening team_test.rb reveals the following:

```ruby
# test/models/team_test.rb

require 'test_helper'

class TeamTest < ActiveSupport::TestCase
  # test "the truth" do
  #   assert true
  # end
end
``` 

The first line references a `test_helper` file. That files has the default configuration for all of the tests. 

```ruby
# test/test_helper.rb

ENV['RAILS_ENV'] ||= 'test'
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'

class ActiveSupport::TestCase
  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical order.
  fixtures :all

  # Add more helper methods to be used by all tests here...
end
```

The first line of code sets the Rails environment to 'test' by default. You can change default behavior by passing in a system environment variable e.g. RAILS_ENV=pre_live.

The `test_helper` also loads all fixtures. Fixtures are a way of providing test data, these files are very similar to using seeds.rb. We will see this in action shortly.

### Run some tests already!

To run a test we can uncomment the sample test in `team_test.rb`:

```ruby
# test/models/team_test.rb

require 'test_helper'

class TeamTest < ActiveSupport::TestCase
  test "the truth" do
    assert true
  end
end
```  


And then use the following command to run the test (will cause an error):

```
rake test
```

That command will have resulted in an error:

```
rake aborted!
ActiveRecord::PendingMigrationError: 

Migrations are pending. To resolve this issue, run:

  bin/rake db:migrate RAILS_ENV=test

```

This is because we have not run the migration to create the teams table. We can follow the instruction and migrate the test database:

```
rake db:migrate RAILS_ENV=test
``` 

To reinforce the fact that we have three different databases, we can compare the migration status of the development database to the test database:

```
rwc $ rake db:migrate:status
Schema migrations table does not exist yet.
rwc $ rake db:migrate:status RAILS_ENV=test

database: /Users/daryn/dev/rwc/db/test.sqlite3

 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20151004143516  Create teams

```

Rails uses the migrations to keep track of which migrations have been applied and that table does not even exist in the development database, and neither does the teams table.

We can now try to run the tests again:

```
rwc $ rake test
Run options: --seed 26769

# Running:

.

Finished in 0.017775s, 56.2577 runs/s, 56.2577 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips

```

The first passing test - boom!

Note: the default rake task is 'test' which means you can run 'rake' instead of 'rake test'.

If we keep the development database up-to-date with migrations we wont need to worry about migrating the test database. This is because Rails will do its best to maintain the test database. This is from the rails guides:

"In order to run your tests, your test database will need to have the current structure. The test helper checks whether your test database has any pending migrations. If so, it will try to load your db/schema.rb or db/structure.sql into the test database. If migrations are still pending, an error will be raised. Usually this indicates that your schema is not fully migrated. Running the migrations against the development database (bin/rake db:migrate) will bring the schema up to date." - http://guides.rubyonrails.org/testing.html#rails-sets-up-for-testing-from-the-word-go





