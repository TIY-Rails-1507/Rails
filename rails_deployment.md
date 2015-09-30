# Deployment


This section covers deploying or releasing an application. We will cover deploying to Heroku. Heroku is a cloud based hosting service.

Heroku has different pricing strategies for different needs:

https://www.heroku.com/pricing

We will be using the free package.

This does mean you need to sign up to Heroku: https://sigAnup.heroku.com/www-home-top

Remember to verify your email.

This section uses bundler. If you want a quick refresher you can read the short descriptions from previous examples:
* https://github.com/TIY-Rails-1507/week-03#bundler
* https://github.com/TIY-Rails-1507/fizzbuzz#gemfile--gemfilelock
* https://github.com/TIY-Rails-1507/rps-tdd#gemfile--gemfilelock

## Rails environments

Rails has support for 3 different environments. These are
* development
* test
* production

The settings for these can be seen in `config/environments`. The files in that directory have defaults that are suitable. For example the development environment will show run time errors on the website, while production has this turned off by default. 

There is another file which is related to environments. This is the `config/database.yml`. That file specifies database settings for different environments. If you wanted to use PostgreSQL in development and test, this is where you would make that change. 

We will be using PostgreSQL as a service from Heroku. Therefore we will not need to make changes here. Heroku will auto generate a 'database.yml' file with the settings that it needs.

Using PostgreSQL as a service means that we don't need to backup the data or manage database upgrades.

Next are the 10 steps to deploy your web application...

## Step 1

There is some gem management which we do need to do. The `Gemfilefile` is located in the root directory, open that for editing. Under source we are going to add the version of Ruby that we have been using. 

We need to specify the version of Ruby that Heroku should use. To see the version you have been using type 'ruby --version' in the console. Once you know this you can add it under sources in the Gemfile.

Terminal:
```
$ ruby --version
ruby 2.2.2p95 (2015-04-13 revision 50295) [x86_64-darwin14]
```

Gemfile:
```ruby
source 'https://rubygems.org'

ruby '2.2.2'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '4.2.3'
# Use sqlite3 as the database for Active Record
gem 'sqlite3'
.
.
.
```

We are going to make a few gem groups, this enables us to use different gems in development, test and production as needed. 


We already have the sqlite3 gem specified in the Gemfile. Move that (with its comment) into the group near the bottom of the file maked as `:development, :test`.

Add the PostgreSQL gem, called 'pg', to a production group.

The Gemfile should contain the following:

```ruby
group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug'

  # Access an IRB console on exception pages or by using <%= console %> in views
  gem 'web-console', '~> 2.0'

  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem 'spring'

  # Use sqlite3 as the database for Active Record
  gem 'sqlite3'
end

group :production do
	# Use pg for PostgreSQL on Heroku 
	gem 'pg'
end

```

Make sure that `gem 'sqlite3'` only appears once in the Gemfile.

Now we need to update the Gemfile.lock. We can do that with the following command:

```
bundle install --without production
```
This will make sure that the Gemfile.lock contains an entry for 'pg' but it won't be installed locally. PostgreSQL (pg) will only be used on Heroku.


## Step 2

We also need to use some 'Heroku gems'. Add the following to the end of the Gemfile:

```
group :production do
	# Use pg for PostgreSQL on Heroku 
	gem 'pg'
	# To enable features such as static asset serving and logging on Heroku
	gem 'rails_12factor'
end
```

Once again run:

```
bundle install --without production
```
<!-- As we have recently run this command, and we have only changed the production group, this should be quick and nothing should be downloaded. -->

## Step 3

Read all of this step before following the instructions.

Install the Heroku toolbelt: https://toolbelt.heroku.com/osx

Follow the instructions on that page.

You should end up with:

```
Authentication successful.
```


If you are using a Mac you can use Homebrew:
```
$ brew install heroku-toolbelt
$ heroku login
.
.
.
```

#### Troubleshooting on a Mac

Here are detailed instructions: 

http://sourabhbajaj.com/mac-setup/Heroku/README.html

#### Troubleshooting on Windows

http://romikoderbynew.com/2012/01/25/getting-started-with-heroku-toolbelt-on-windows/

#### Troubleshooting on Ubuntu

http://askubuntu.com/questions/556685/how-to-download-and-install-heroku

## Step 4

Heroku uses Git for deploying. Make sure that all your changes are committed locally - you don't need to push to GetHub.

You can verify a clean branch with:

```
$ git status
On branch master
nothing to commit, working directory clean
```

## Step 5

### Create the app on Heroku

Make sure you are in the root application of your rails app. To create an application (repo) on Heroku run:

```
$ heroku create
```

This should give output similar to:

```
Creating afternoon-waters-9192... done, stack is cedar-14
https://afternoon-waters-9192.herokuapp.com/ | https://git.heroku.com/afternoon-waters-9192.git
Git remote heroku added
```

To verify that this command worked use `git remote -v`:

```
$ git remote -v
heroku	https://git.heroku.com/afternoon-waters-9192.git (fetch)
heroku	https://git.heroku.com/afternoon-waters-9192.git (push)
origin	https://github.com/TIY-Rails-1507/questionable2.git (fetch)
origin	https://github.com/TIY-Rails-1507/questionable2.git (push)

```

Notice a new remote has been added that points to `git.heroku.com`.

## Step 6

### Push the code to Heroku

Now we need to do is push the code to that remote location.

```
$ git push heroku master
Counting objects: 215, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (199/199), done.
Writing objects: 100% (215/215), 96.91 KiB | 0 bytes/s, done.
Total 215 (delta 71), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote: 
remote: -----> Removing .DS_Store files
remote: -----> Ruby app detected
remote: -----> Compiling Ruby/Rails
...
...
...
remote:        Cleaning assets
remote:        Running: rake assets:clean
remote: 
remote: ###### WARNING:
remote:        No Procfile detected, using the default web server (webrick)
remote:        https://devcenter.heroku.com/articles/ruby-default-web-server
remote: 
remote: -----> Discovering process types
remote:        Procfile declares types -> (none)
remote:        Default types for Ruby  -> console, rake, web, worker
remote: 
remote: -----> Compressing... done, 29.9MB
remote: -----> Launching... done, v5
remote:        https://afternoon-waters-9192.herokuapp.com/ deployed to Heroku
remote: 
remote: Verifying deploy.... done.
To https://git.heroku.com/afternoon-waters-9192.git
 * [new branch]      master -> master
$ 
```

There may be a warning about no Procfile, we can ignore this. Verify that no other errors occurred. 

## Step 7

### Run the migrations

Next we need to run the migrations so that the PostgreSQL database is created and updated.

```
$ heroku run rake db:migrate
Running `rake db:migrate` attached to terminal... up, run.6451
   (15.7ms)  CREATE TABLE "schema_migrations" ("version" character varying NOT NULL) 
   (10.2ms)  CREATE UNIQUE INDEX  "unique_schema_migrations" ON "schema_migrations"  ("version")
  ActiveRecord::SchemaMigration Load (2.8ms)  SELECT "schema_migrations".* FROM "schema_migrations"
Migrating to CreateQuestions (20150817190340)
...
...
...
```

### Make sure the app is up and running

Next we need to make sure that one 'dyno' is running:

```
$ heroku ps:scale web=1
Scaling dynos... done, now running web at 1:Free.
```

For more information on Dynos read here: https://devcenter.heroku.com/articles/dynos#dynos


Use the following to verify that the Dyno is running:

```
$ heroku ps
=== web (Free): `bin/rails server -p $PORT -e $RAILS_ENV`
web.1: up 2015/08/30 18:11:01 (~ 10m ago)
```

## Step 8

### Visit the application

```
$ heroku open
```

You may see your splendid application or you may see `The page you were looking for doesn't exist.`.

If you get that error, it is likely that you simply need to correct the URL e.g. add on '/hotels'

If you did get this error, make sure you set a `root route` in routes.rb. There are comments within that file explaining how to do that. (And Daryn knows...)


## Step 9

### Update the app

To re-deploy, simply use Git to push another version of the app to Heroku.

## Step 10

##### Ta-da! :) 





