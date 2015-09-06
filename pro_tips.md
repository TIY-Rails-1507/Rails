
### Sublime config for Mac 
If you are a Mac, you can open folders\files in Sublime from the terminal. 

Use:
subl "dir_name"
subl "file_name"

If that does not work try:

sudo ln -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl /usr/bin/subl



### Gemsets for RVM

If you are using gemsets, this page is useful: https://rvm.io/gemsets



### Duplicate entries in Rails console

If you are getting duplicate entries in the console or Rails logs, it might be due to the `rails_12factor` gem.

The solution is to follow these steps:
* Manually remove the gem: gem uninstall rails_12factor
* move this gem into the production group in the gem file
* run `bundle install` 

For more information see here: http://stackoverflow.com/a/30286154/259477

