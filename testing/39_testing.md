# Testing in Rails

This section covers the fundamentals of testing. Automatically testing an application gives us a high level of confidence that we have not introduced defects. 

Testing that previous functionality has not been effected is known as regression testing. This can take days, or even weeks, for teams that do not have automated tests. 

Technically it is better to test an application from the user interface layer, and work your way down. This is known as 'outside-in testing' as is promoted by a development technique known as Behavior Driven Development - http://dannorth.net/introducing-bdd/. 

As this is our first exposure to testing in Rails it may be easier to grasp starting from the model and working our way up the layers.

## Setup

In this section we will start with a new application called RWC - short for Rugby World Cup. RWC will list the teams in the tournament. 

Each team will have a ranking. The ranking is a percent where the higher the number the better The application will try to predict a winner of a game, based on the raking.

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


### Note

These notes cover testing models, controllers, and views. There is plenty more to know about testing e.g. testing routes. For a deeper look into testing see the Rails Guides - http://guides.rubyonrails.org/testing.html





