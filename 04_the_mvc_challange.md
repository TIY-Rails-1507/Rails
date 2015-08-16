# The MVC challenge
* Use the the hotel model in the hotels controller to fetch all the hotels in the database
* Check that the view is displaying this correctly

#### Hashes as parameters and the new syntax

Rails has a way of displaying a number as a currency
* The method: number_to_currency(hotel.price)
  * http://api.rubyonrails.org/classes/ActionView/Helpers/NumberHelper.html
* As described in the docs, the method signature is:
  * number_to_currency(number, options = {})
  * This means that the last parameter is a hash
* As explained in the documentation one of the options is unit
  * :unit - Sets the denomination of the currency (defaults to “$”).
* Another option is precision
  * :precision - Sets the level of precision (defaults to 2).

You can experiment with this method in the rails console:
```
rails console
helper.number_to_currency(10)
```
Only use 'helper' when in the console. Within the View we don't need to specify 'helper'.

To use this method with defaults use:
```ruby
helper.number_to_currency(10)
```

To use this method with some options use:
```ruby
opt = { :unit => '£', :precision => 3 }
helper.number_to_currency(10, opt)
```
Next we can in-line the hash
```ruby
helper.number_to_currency(10, { :unit => '£', :precision => 3 })
```
In Ruby if the last parameter is a hash, then the parentheses are not required

```ruby
number_to_currency(10, :unit => '£', :precision => 3)
```
Using the new hash syntax for symbols
```ruby
helper.number_to_currency(10, unit: '£', precision: 3 )
```
