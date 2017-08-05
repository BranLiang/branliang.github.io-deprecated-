---
layout:     post
title:      "Just things you may not know about Ruby"
---

## Introduction  

Recently read a book called '[Confident Ruby](https://www.confidentruby.com/)' by Avdi Grimm. This is a book about how to write methods in ruby properly and I found many interesting stuff there which I did not pay attention before. So I would like to share my learning here to you guys and hope you could benefit a little here.

## Let the game begin!  

### to_int  
```ruby
class A
  ...
  def to_int
    @index
  end
end
# You can also add this method to struct, which is a simplified form of class.
a_instance = A.new(blabla)
some_array[a_instance]
```  
```ruby
class B
  ...
  def to_str
    "world"
  end
end
b_instance = B.new(blabla)
puts "hello " + b # => hello world
```
The code above will work! The array can accept object as parameter rather than common integer index! Why? Because whenever the array calls, the parameter will implicitly call `.to_int`, therefore if I serve the object with `to_int` method, this will just work fine.  
__Adding ruby conversion method to class will help, because ruby will implicitly call them when needed__

### deal with units  
```ruby
class Meters
  ...
  def to_meters
    self
  end
end
class Feet
  ...
  def to_meter
    Meters.new((value * 0.3048).round)
  end
end
# value is safe to operate without assertions, as long as you call each a .to_meter method
value_a.to_meter + value_b.to_meter
```  

### Array VS .to_a/to_ary  
```ruby
# not working!
123.to_a / 123.to_ary # => undefined method `to_ary` for 123:Fixnum
# it works!
Array(123) # => [123]
```
__If you by no means want to convert the input to be array or integer, use `Integer(), Array()...`__  

### define conversion function  
```ruby
class A
  ...
  private
    def A(a)
      case a
      when A then a
      else
        ....
      end
    end
```
Just a object conversion template. When you need the parameter to be a specific object, you can simply define a conversion function there.  

### ruby interpolation format  
```ruby
"%s, %s" % ['hello', 'world'] # => hello, world
"%<a>s, %<b>s" % [a: 'hello', b: 'world'] # => hello, world
```  

### new class pattern  
when one method need to deal with multiple input situations, consider separating the special case into its own class and serve it the same method name but with different implementation.  

```ruby
class A
  def do_something(msg)
    case msg
    when Special
      @something.do_something_special
    else
      @something.do_something_normal
    end
  end
end
# another pattern
class A
  class B
    def do_something_normal
    end
  end

  def initialize(msg)
    @something = case msg
    when Special then B.new(blabla)
    else msg
    end
  end
end
```  

__Generally, the separation is performed when initialized, and whatever the instance object is, they all share the same function name, but just different content__  

### fetch for Hash  
```ruby
hash_a = {a: 1, b:2}
hash_a.fetch(:c) # => Error!! Which is good, because we know there is something wrong here.
hash_a[:c] # => nil, bad! It just fail silently!

# also you can assign default value to a hash using fetch
hash_a.fetch(:c, 15) # => 15
# another way
hash_a.fetch(:c) {15} # => 15
```  
Another good practice is when using fetch, it is always good to give a error symbol when the key is not found. Like follows:  

```ruby
value = hash.fetch(:key) { :key_not_set }
```

### Guest User situation  
Normal way to declare the current_user, and the way to user it.  

```ruby
def current_user
  if session[:user_id]
    User.find(session[:user_id])
  end
end
# current_user usage, for each place you need to check if the current is present or not?
def method_a
  if current_user
    do_something
  else
    do_else
  end
end
```  

Another way to do the same check for current_user is to create a separate GuestUser class. This way the nil check can be avoided.

```ruby
class GuestUser
  def initialize(session)
    @session = session
  end

  def name
    "Anonymous"
  end
end

class User
  def current_user
    if session[:user_id]
      User.find(session[:user_id])
    else
      GuestUser.new(session)
    end
  end
end
```  

### deal with optional information  
```ruby
# prefer
def method
  optional_value = unreliable_calculation || default_value
  use_it(optional_value)
end
# not preferred way
def method
  optional_value = unreliable_calculation
  if optional_value
    use_it(optional_value)
  end
  # or
  begin
    optional_value = unreliable_calculation
    use_it(optional_value)
  rescue NoMethodError
  end
end
```  

### add callback to methods  
```ruby
def method_with_callback(bla, bla, &callback)
  blabla
  unless blablabla
    callback.call(params)
  end
end
method_with_callback(bla, bla) do |params|
  ...
end
```  

### "" VS nil  
```ruby
# prefer return value
""
# not preferred value
nil
```  

### Top-level rescue clause  
You probably have been used to the normal begin rescue end syntax to dealing errors. However, you can also use the top-level rescue clause, which is only just one line.

```ruby
def method
  begin
    ...
  rescue
    ...
  end
end
# is equal to the usage before
def method
  # happy situations
rescue
  # sad situations
end
```
