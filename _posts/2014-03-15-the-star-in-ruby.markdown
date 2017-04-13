---
layout: post
title: star in ruby
date: 2014-03-15 00:11
comments: true
categories: Ruby, Programming, Code
published: true
---

I have seen the Ruby's `*` operator used at several places in different
ways and often times I get confused the way it is used. Finally I decided to learn more about it and here are few interesting roles of `*` operator that I found in Ruby.
It can be used to multiply, repeat, copy or as a splat operator.


---

# Multiplication

This is the most obvious role of `*`. It's the multiplication operator. `2 * 2 = 4`

Or an instance method of **Fixnum** class to perform multiplication. `2.*(2) = 4`


# Repetition

Several classes in Ruby defines `*` operator as a Repetition method.

* **Array** class defines **\*** for two different possible type of parameters.

{% highlight ruby %}
ary * int -> new_ary
ary * str -> new_str
{% endhighlight %}

For string type it acts as a `join` method. Examples:

{% highlight ruby %}
[1,2,3] * 3 # => [1, 2, 3, 1, 2, 3, 1, 2, 3]

[1,2,3] * ',' # => "1,2,3"
{% endhighlight %}

* **String** class defines **\*** as a **copy** operator. It returns a new string
containing **n** copies of the string. For example:

{% highlight ruby %}
"ho! " * 3 # => "ho! ho! ho! "
{% endhighlight %}

Note that **`String * str`** is not defined.

# Splat

Another less known use of `*` is as a **splat** operator. Consider
following example where we want to convert options_array to
options_hash.

{% highlight ruby %}
options_array = [:first_name, "Ankit", :last_name, "Goyal"]
# to
options_hash = {:first_name => "Ankit", :last_name => "Goyal"}
{% endhighlight %}

You will see that [`[]`]( http://www.ruby-doc.org/core-2.1.0/Hash.html#method-c-5B-5D ) method in **Hash** class takes [key, value, ...] as an argument and
creates a Hash for you. For example:

{% highlight ruby %}
> Hash[1,2,3,4]
# {1 => 2, 3 => 4}
{% endhighlight %}

So how can we use `[]` method to get the required hash?  You can see in
documentation that `[]` doesn't
take array as a parameter or you can try running it. It will return you an empty hash with lot of warnings. So we can't call `Hash[options_array]` directly. 

* We can use splat `*` operator to convert our array into arguments. If
you call `[]` with `*options_array` as a parameter, you'll get the
required hash back.

{% highlight ruby %}
options_hash = Hash[*options_array]
# {:first_name => "Ankit", :last_name => "Goyal"}
{% endhighlight %}

So splat converts `*options_array` to an argument list. Let's see some
other interesting uses of splat operator.

**Splat** can be used to convert an array to argument list as shown above. You can call methods by dynamically generating arguments with minimal
amount of code. For example, if you have a **string** with ',' separated
values of arguments.


{% highlight ruby %}
def do_it(arg1, arg2, arg3)
    puts "#{arg1} #{arg2} #{arg3}"
end

arguments = "arg1,arg2,arg3"
{% endhighlight %}

You can call `do_it` method with `arg1`, `arg2` and `arg3` as arguments
using splat like this:

{% highlight ruby %}
do_it(*arguments.split(','))
# => "arg1 arg2 arg3"
{% endhighlight %}

* You can also convert list of arguments back to array using splat. For
example:

{% highlight ruby %}
*argument_array = 1, 2, 3
> argument_array
# => [1, 2, 3]
{% endhighlight %}

**Note** that you don't need to use splat for the above code to work. 

{% highlight ruby %}
argument_array = 1, 2, 3
> argument_array
# => [1, 2, 3]
{% endhighlight %}

is equally valid. But it's good to know that you can do it using splat
also.

However, a case where splat comes in handy is to define multiple
variables. Let's see an example:

{% highlight ruby %}
a, b, *c = 1, 2, 3, 4, 5, 6
{% endhighlight %}

Above fact is mostly used in defining methods with optional arguments.
This is one of the most common uses of the splat operator.

{% highlight ruby %}
def do_it2(arg1, arg2, *options)
    puts "#{arg1} - #{arg2} - #{options}"
end

do_it2(1, 2, 3, 4, 5, :cool, "abc")
# => 1 - 2 - [3, 4, 5, :cool, "abc"]
{% endhighlight %}

It is not necessary to have the splat argument at the end in the
method! For example definition like this is equally valid:

{% highlight ruby %}
def do_it3(*options, arg1, arg2)
    puts "#{arg1} - #{arg2} - #{options}"
end

# You must call the method with 2 or more arguments.
> do_it3(1,2, hello)
# => 2 - hello - [1]

{% endhighlight %}

However it's invalid to have more than 1 splat arguments.

{% highlight ruby %}
# This is not valid
def do_it_invalid(*a, *b)
end
{% endhighlight %}

* You can also use splat operator to convert Hash into an array of
  arrays. For example:

{% highlight ruby %}

options_hash = {:first_name => "Ankit", :last_name => "Goyal"}
options_array_of_array = *options_hash
# => [[:first_name, "Ankit"], [:last_name, "Goyal"]]

{% endhighlight %}

* You can use splat operator to un-nest arrays. For example:

{% highlight ruby %}

a = [*[1,2], 3, 4]

> a
# => [1, 2, 3, 4]
{% endhighlight %}

Splat operator can possibly be used in several creative ways. If you
know some other way that splash could be used feel free to suggest it in comments.


## References/Further Reading:

1. http://endofline.wordpress.com/2011/01/21/the-strange-ruby-splat/
2. http://www.jacopretorius.net/2012/01/splat-operator-in-ruby.html
3. http://www.ruby-doc.org/



