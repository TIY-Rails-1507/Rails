# Testing views

In this section we check that the correct HTML is being created. These are actually an extension of controller tests, where we validate that specific HTML elements are being created.


Let's check the value of the header on the page. We will be adding a header in an H1 tag, which means that the test will look as follows:

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

  test "should display RWC Rankings as the title" do
    get :index
    assert_select 'h1', "RWC Rankings"
  end

end
```

This produces the following error:

```ruby
<RWC Rankings> expected but was
<Teams#index>..
Expected 0 to be >= 1.
```
It seems to have found an h1 element with the value of Teams#index. Openging up app/views/teams/index.html.erb explains why:

```html
<h1>Teams#index</h1>
<p>Find me in app/views/teams/index.html.erb</p>
```
Let's change that to the following:

```html
<h1>RWC Rankings</h1>
```

We could check that all three teams are being displayed. Let's assume that each team is displayed in a div with a class of 'team'. Then the test could look as follows:


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

  test "should display RWC Rankings as the title" do
    get :index
    assert_select 'h1', "RWC Rankings"
  end

  test "should display 3 teams" do
    create(:team)
    create(:team, name: "Australia", rank: 86.75)
    create(:team, name: "South Africa", rank: 82.66)

    get :index
    assert_select '.team', 3
  end

end

```
The error message is fairly clear:

```
  1) Failure:
TeamsControllerTest#test_should_display_3_teams [/Users/daryn/dev/rwc/test/controllers/teams_controller_test.rb:28]:
Expected exactly 3 elements matching ".team", found 0..
Expected: 3
  Actual: 0

9 runs, 18 assertions, 1 failures, 0 errors, 0 skips
```

Let's write the code for that:

```html
<h1>RWC Rankings</h1>

<%= @teams.each do |team| %>
  <div class="team">
    
  </div>
<% end %>
```

While this gets the test to pass, it does not show any information. We can make the test a bit more detailed:

```ruby
test "should display 3 teams" do
  new_zealand = create(:team)
  australia = create(:team, name: "Australia", rank: 86.75)
  south_africa = create(:team, name: "South Africa", rank: 82.66)
  teams = [new_zealand, australia, south_africa]
  
  get :index

  teams.each_with_index do | team, index |
    assert_select ".team:nth-of-type(#{index + 1})" do | team_element |
      assert_select team_element, "span[name='team_name']", team.name
      assert_select team_element, "span[name='team_rank']", team.rank.to_s
    end
  end
end
```
Note the use of `team.rank.to_s`. Without that `assert_select` would have assumed we were giving it a number of elements to find. However in this case we would have given it a fraction. This would have translated to - find elements with this css selector, "span[name='team_rank']" and there should be 92.86 of them! It is no wonder that if you remove the `to_s` you get the following error: 

```
ArgumentError: I don't understand what you're trying to match
```

Running the tests now results in:

```ruby
TeamsControllerTest#test_should_display_3_teams [/Users/daryn/dev/rwc/test/controllers/teams_controller_test.rb:34]:
Expected at least 1 element matching "span[name='team_name']", found 0..
Expected 0 to be >= 1.

10 runs, 19 assertions, 1 failures, 0 errors, 0 skips
```

The following code will get this test to pass:

```ruby
<h1>RWC Rankings</h1>

<%= @teams.each do |team| %>
  <div class="team">
    <p> 
      Team: <span name="team_name"><%= team.name %></span>
    </p>
    <p>
      Ranking: <span name="team_rank"><%= team.rank %></span>
    </p>
  </div>
<% end %>
```

Read more about testing views here: http://guides.rubyonrails.org/testing.html#testing-views

#### Warning! warning! 

That test above is fairly complex, it would be worth spending time on simplifying it. Even if it is just removing the list of teams and the outer loop...

