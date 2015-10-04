# Test Data

In this section we see how to inject the database with known data so that we can perform additional tests. There are many ways of doing this, we will look at two approaches.


## Fixtures

Fixtures are the built in way of handling test data. They allow us to write sample test data in a YML file, this data will then be loaded into the test data when the the tests are run. 


Rails has already generated a template fixture file for us:

```ruby
# test/fixtures/teams.yml

# Read about fixtures at http://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html

one:
  name: MyString
  rank: 9.99

two:
  name: MyString
  rank: 9.99
```

We can modify this to contain real test data:

```
# test/fixtures/teams.yml
# Read about fixtures at http://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html

NewZealand:
  name: "New Zealand"
  rank: 92.86

Wales:
  name: "Wales"
  rank: 87.31

Australia:
  name: "Australia"
  rank: 86.75

Ireland:
  name: "Ireland"
  rank: 84.40

SouthAfrica:
  name: "South Africa"
  rank: 82.66
``` 

According to the Rails guides:
Rails by default automatically loads all fixtures from the test/fixtures folder for your models and controllers test. Loading involves three steps:

* Remove any existing data from the table corresponding to the fixture
* Load the fixture data into the table
* Dump the fixture data into a variable in case you want to access it directly


This means we can use this data within the tests. Let's implement a method which predicts a winner based on the ranking. We first begin with a failing test:

```ruby
# test/models/team_test.rb

require 'test_helper'

class TeamTest < ActiveSupport::TestCase
  test "should not save a team without a name" do
    team = Team.new(rank: 50)
    assert_not team.save
    assert_equal ["can't be blank"], team.errors.messages[:name]
  end

  test "should not save a team without a rank" do
    team = Team.new(name: "England")
    assert_not team.save
    assert_equal ["can't be blank", "is not a number"], team.errors.messages[:rank]
  end

  test "rank should be a number" do
    team = Team.new(name: "England", rank: "not a number")
    assert_not team.save
    assert_equal ["is not a number"], team.errors.messages[:rank]
  end

  test "rank should be greater than 0" do
    team = Team.new(name: "England", rank: -1)
    assert_not team.save
    assert_equal ["must be greater than 0"], team.errors.messages[:rank]
  end

  test "rank should be less than 100" do
    team = Team.new(name: "England", rank: 100)
    assert_not team.save
    assert_equal ["must be less than 100"], team.errors.messages[:rank]
  end

  test "should not expect to beat a team with a higher rank" do
    australia = teams(:Australia)
    new_zealand = teams(:NewZealand)

    assert_not australia.expected_to_beat new_zealand 
  end
end
```

The last test is using data which is being stored in the database, these fields have IDs they are not 'new records'.

This last test will fail as we don't have the method 'expected_to_beat'. In true test driven development (TDD) style we will only write the code that we need:

```ruby
class Team < ActiveRecord::Base
  validates :name, :rank, presence: true
  validates :rank, numericality: { greater_than: 0, less_than: 100 }

  def expected_to_beat other_team
    false
  end
end
``` 

This causes the tests to pass, but obviously we need to flesh out the method with more logic. So, we write another failing test:

```ruby
test "should expect to beat a team with a lower rank" do
  australia = teams(:Australia)
  new_zealand = teams(:NewZealand)

  assert new_zealand.expected_to_beat australia 
end
```
This results in 1 failing test. We can get both of these tests to pass with the following change:

```ruby
class Team < ActiveRecord::Base
  validates :name, :rank, presence: true
  validates :rank, numericality: { greater_than: 0, less_than: 100 }

  def expected_to_beat other_team
    rank > other_team.rank
  end
end
```

Running the tests now produces the following results:

```
rwc $ rake
Run options: --seed 42895

# Running:

.......

Finished in 0.033172s, 211.0242 runs/s, 361.7558 assertions/s.

7 runs, 12 assertions, 0 failures, 0 errors, 0 skips
```

Fixtures can be quite complex, they can even include lines if ERB when required. To learn more about fixtures see these links  

* http://guides.rubyonrails.org/testing.html#the-low-down-on-fixtures
* http://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html


## Factories

While Fixtures seem convenient they tend to become unmanageable. This happens when we start cross referencing the data e.g. a team has many players. It is also not easy to see what the values are in the tests e.g. in the test above why is New Zealand expected to beat Australia? 

For these reasons and many more, it is highly recommended to use factories over fixtures. 

Before continuing comment out the fixtures, regular Ruby comments will do.


In these examples we will use a gem called Factory Girl. Add the following to your Gemfile:

```
gem "factory_girl_rails"
```

That can go into the a group for :development and :test.

Then install the gem:

```
bundle install
```

Add the mixin into the test class:

```ruby
# test/models/team_test.rb

require 'test_helper'

class TeamTest < ActiveSupport::TestCase
  include FactoryGirl::Syntax::Methods
...
...
...
```

Next we need to define a factory:

```ruby
# test/models/team_test.rb

require 'test_helper'

class TeamTest < ActiveSupport::TestCase
  include FactoryGirl::Syntax::Methods

  FactoryGirl.define do
    factory :team do
      name "New Zealand"
      rank 92.86 
    end
  end

  test "should not save a team without a name" do
    team = Team.new(rank: 50)
```

That defines a factory with default values. 

We can now use this factory in the tests. For this we will change these tests from this:

```ruby

test "should not expect to beat a team with a higher rank" do
  australia = teams(:Australia)
  new_zealand = teams(:NewZealand)

  assert_not australia.expected_to_beat new_zealand 
end

test "should expect to beat a team with a lower rank" do
  australia = teams(:Australia)
  new_zealand = teams(:NewZealand)

  assert new_zealand.expected_to_beat australia 
end

```

To this:

```ruby

test "should not expect to beat a team with a higher rank" do
  new_zealand = create(:team)
  australia = create(:team, name: "Australia", rank: 86.75)

  assert_not australia.expected_to_beat new_zealand 
end

test "should expect to beat a team with a lower rank" do
  new_zealand = build(:team)
  australia = build(:team, name: "Australia", rank: 86.75)

  assert new_zealand.expected_to_beat australia 
end

```

Here is a walk through of the key lines:

```
new_zealand = create(:team)
``
This creates a new team in the database with the default values specified in the factory e.g. team name of New Zealand

```ruby
australia = build(:team, name: "Australia", rank: 86.75)
```
This creates a new team in the database, with the factories default overridden. 

```ruby
new_zealand = build(:team)
```
This creates a new team with the factories default values, but the team has not been saved to the database.

For more options see here: https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md#using-factories

It would be good to have this factory in a separate file so that other tests can re-use it. According to the factory girl documentation, Factories can be defined anywhere, but will be automatically loaded if they are defined in files at the following locations:
* test/factories.rb
* spec/factories.rb
* test/factories/*.rb
* spec/factories/*.rb

Let's move this factory into it's own file, create a new file at this location test/factories.rb and move the factory into that file.

```ruby
# test/factories.rb

FactoryGirl.define do
  factory :team do
    name "New Zealand"
    rank 92.86 
  end
end
```

Check that all the tests are still passing...

#### References 
* https://semaphoreci.com/blog/2014/01/14/rails-testing-antipatterns-fixtures-and-factories.html 
* https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md
