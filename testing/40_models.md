## Testing the model

The current test simply checks that 'true' is 'true'. Let's change that to a more meaningful example. Let's assume we should not be able to save a team without a name. 

First we right the test:

```ruby
# test/models/team_test.rb

require 'test_helper'

class TeamTest < ActiveSupport::TestCase
  test "should not save a team without a name" do
    team = Team.new
    assert_not team.save
  end
end
```

Running `rake test` should result in one failure. Next we implement the validation:

```ruby
class Team < ActiveRecord::Base
  validates :name, presence: true
end
```

Running `rake test` should show one test passing. 

We could make the test more specific to check the actual validation error:

```ruby
# test/models/team_test.rb

require 'test_helper'

class TeamTest < ActiveSupport::TestCase
  test "should not save a team without a name" do
    team = Team.new
    assert_not team.save
    assert_equal ["can't be blank"], team.errors.messages[:name]
  end
end
```

Similarly for the rank:

```ruby
# test/models/team_test.rb

require 'test_helper'

class TeamTest < ActiveSupport::TestCase
  test "should not save a team without a name" do
    team = Team.new
    assert_not team.save
    assert_equal ["can't be blank"], team.errors.messages[:name]
  end

  test "should not save a team without a rank" do
    team = Team.new
    assert_not team.save
    assert_equal ["can't be blank"], team.errors.messages[:rank]
  end
end
```

Notice the second test only failed when checking the exact error message and not on the first assert which checked if the team was saved. This is because we are not providing a team name. A better way to write these tests is as follows:

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
    assert_equal ["can't be blank"], team.errors.messages[:rank]
  end
end
```

Implementing the production code:

```
class Team < ActiveRecord::Base
  validates :name, :rank, presence: true
end
```

Results from the tests:

```ruby
2 runs, 4 assertions, 0 failures, 0 errors, 0 skips
```

Next we can add a validation that verifies that the rank is a number:

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
    assert_equal ["can't be blank", "is not a number"], team.errors.messages[:rank] # <-- updated!
  end

  test "rank should be a number" do
    team = Team.new(name: "England", rank: "not a number")
    assert_not team.save
    assert_equal ["is not a number"], team.errors.messages[:rank]
  end
end
```
This test fails, add the following to make it pass:

```ruby
class Team < ActiveRecord::Base
  validates :name, :rank, presence: true
  validates :rank, numericality: true
end
```

Now we want to make sure that the rank is greater than 0 and less than 100. Tests for these rules could look like this:

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
end
```

Making them pass:

```ruby
class Team < ActiveRecord::Base
  validates :name, :rank, presence: true
  validates :rank, numericality: { greater_than: 0, less_than: 100 }
end
```

There is an alternative way of implementing this validation, let's try this:

```ruby
class Team < ActiveRecord::Base
  validates :name, :rank, presence: true
  validates :rank, numericality: true
  validates_inclusion_of :rank, in: 0...100, message: "the rank must be between 0 and 100"
end

```

If you choose that alternative then you will need to update the tests...