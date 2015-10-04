# Controller Tests

Controller tests are also known as functional tests. Once again, Rails will help us by creating template tests when we perform a generate command.

Do the following:

```
$ rails generate controller teams index
```

The output

```
create  app/controllers/teams_controller.rb
       route  get 'teams/index'
      invoke  erb
      create    app/views/teams
      create    app/views/teams/index.html.erb
      invoke  test_unit
      create    test/controllers/teams_controller_test.rb
      invoke  helper
      create    app/helpers/teams_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/teams.coffee
      invoke    scss
      create      app/assets/stylesheets/teams.scss
```

Note there was a route added, that is shown in the first line of the output. 

With regards to testing, teams_controller_test.rb is the important file.

```ruby
# test/controllers/teams_controller_tests.rb

require 'test_helper'

class TeamsControllerTest < ActionController::TestCase
  test "should get index" do
    get :index
    assert_response :success
    assert_not_nil assigns(:teams)
  end

end
```

The `get` method call actually makes a web request and populates the results into the response. `assert_response` checks the the expected response header was sent. It could be one of these: success, redirect, missing, or error. You can also check for explicit error codes e.g. assert_response(501).

We would like this index action to set a list of teams to `@teams`. Using a test first approach we can modify the test to the following:

```ruby
# test/controllers/teams_controller_tests.rb

require 'test_helper'

class TeamsControllerTest < ActionController::TestCase
  test "should get index" do
    get :index
    assert_response :success
    assert_not_nil assigns(:teams), "The teams were not set"
  end

end
```
Running the tests will cause this one to fail. The custom message helps to diagnose the issue.

```
 1) Failure:
TeamsControllerTest#test_should_get_index [/Users/daryn/dev/rwc/test/controllers/teams_controller_test.rb:9]:
The teams were not set.
Expected nil to not be nil.
```


To fix this we need add the following to the controller:

```ruby
class TeamsController < ApplicationController
  def index
    @teams = Team.all
  end
end
```

If we wanted to we could check the length of the teams list:

```ruby
# test/controllers/teams_controller_tests.rb

require 'test_helper'

class TeamsControllerTest < ActionController::TestCase
  test "should get index" do
    get :index
    assert_response :success
    teams = assigns(:teams)
    assert_not_nil teams, "The teams were not set"
    assert_equal 3, teams.count
  end

end
```

This fails because we are not creating any teams in the database. To do that we need to use the factory.

```ruby
# test/controllers/teams_controller_tests.rb

require 'test_helper'

class TeamsControllerTest < ActionController::TestCase
  include FactoryGirl::Syntax::Methods
  
  test "should get index" do
    create(:team)
    create(:team, name: "Australia", rank: 86.75)
    create(:team, name: "South Africa", rank: 82.66)

    get :index
    assert_response :success
    teams = assigns(:teams)
    assert_not_nil teams, "The teams were not set"
    assert_equal 3, teams.count
  end

end
```

This test is not meant to serve as an example of best practice, it is simply meant to show how you can use Factory Girl in a controller test if needed. 


We can test that the correct template is being returned. In this case we expect the `index` template to be returned. To verify this in a test we can make the following addition:

```ruby
# test/controllers/teams_controller_tests.rb

require 'test_helper'

class TeamsControllerTest < ActionController::TestCase
  include FactoryGirl::Syntax::Methods

  test "should get index" do
    create(:team)
    create(:team, name: "Australia", rank: 86.75)
    create(:team, name: "South Africa", rank: 82.66)

    get :index
    assert_response :success
    teams = assigns(:teams)
    assert_not_nil teams, "The teams were not set"
    assert_equal 3, teams.count
    assert_template :index
  end

end
```

This has been a brief look at controller testing. For more details have a look at the Rails guides - http://guides.rubyonrails.org/testing.html#functional-tests-for-your-controllers

