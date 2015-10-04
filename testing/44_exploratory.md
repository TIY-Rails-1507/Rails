# Exploratory Testing

Test automation is immensely valuable. It gives us fast feedback and helps to do repetitive regression testing. However it does not replace the need for manual exploratory testing.

Let's have a quick manual check to see if the site looks as it should. We are going to want some teams in the database. We can use seeds.rb for that:

```ruby

# db/seeds.rb
Team.create([{name: "New Zealand", rank: 92.89}, {name: "Wales", rank: 87.31}, {name: "Australia", rank: 86.75}, {name: "Ireland", rank: 84.40}, {name: "South Africa", rank: 82.66}])
```

Make sure the migrations have been run on the development database:

```
$ rake db:migrate
```

Seed the database:

```
$ rake db:seed
```

Start the server and go to: http://localhost:3000/teams/index

### Ooops?

Did you see any errors? If so, fix-em! 

Also ask yourself, can this be fixed with by writing a test first? 