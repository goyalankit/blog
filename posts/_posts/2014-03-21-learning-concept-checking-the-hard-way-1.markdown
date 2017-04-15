---
layout: post
title: "Learning concept checking the hard way - P1"
date: 2014-03-21 15:27
comments: true
categories: code programming cpp templates concept-checking
published: true
---

# What is a concept in C++?

> A concept is a set of requirements (valid expressions, associated types, semantic invariants, complexity guarantees, etc.) 
that a type must fulfill to be correctly used as arguments in a call to a generic algorithm

For example if you write your own generic implementation for
`BinarySearchTree`. It should basically work for any type for which `<`
operator is defined. As long as you have that condition satisfied you
can build a binary search tree for that type. For instance, `<` is defined for
`int`, `double`. However it's not defined for `complex` numbers. C++ has
no explicit mechanism for representing concepts.

Consider the following possible way 
you may write your generic Binary Search Tree.

{% highlight cpp %}
#include<iostream>

template <typename T>
class BinarySearchTree{
    public:
    // dummy method to insert a new node to the tree
    void insert(T value, T parent){
        // calling `<` operator. Compilation will fail if it doesn't exist
        // for a type.
        if(value < parent){
            std::cout << "Less than operator exists" << std::endl;
            //insert to the left of binary tree.
        }
    }
};

int main(int argc, char* argv[]){
    // instantiating template with int
    BinarySearchTree<int> bint;
    bint.insert(1,2);
    // instantiating template with double
    BinarySearchTree<double> bdouble;
    bdouble.insert(1.23,3.13);

    return 0;
}

{% endhighlight %}


{% highlight sh %}
>> g++ b.cpp -o btree
>> ./btree
Less than operator exists
Less than operator exists
{% endhighlight %}

Awesome, so it works for both double and int, since `<` is defined for
them. What happens if you try it with complex numbers? let's see:

{% highlight cpp %}

//include the complex lib
#include <complex>

// In main function
// instantiating template with complex
BinarySearchTree<std::complex<int> > bcomplex;
bcomplex.insert(std::complex<int>(1,2), std::complex<int>(4,5));

{% endhighlight %}

If you try to run your `BinarySearchTree` implementation for `complex`
numbers you will get a compile time error. So the question is how do you
check if a class has a certain method defined or not. Above Example is a
trivial example to show you the use case, the error could be buried deep 
into your code and may not represent the actual reason for
failure. Boost documentation gives a [good example]( http://www.boost.org/doc/libs/1_55_0/libs/concept_check/concept_check.htm ).

In short it would be good to have a `has_less` method for a generic type 
that determines if the given type has a `<` method defined or not. Using
this method you can give your user a more meaningful error.

Note that you can use concept checking provided by boost library. You'd
need to do something like this:

{% highlight cpp %}
#include <boost/concept_check.hpp>

//inside class
BOOST_CLASS_REQUIRE(T, boost, LessThanComparableConcept);

{% endhighlight %}

For more info: visit http://www.boost.org/doc/libs/1_55_0/libs/concept_check/using_concept_check.htm

In the [next post](http://goyalankit.com/blog/2014/03/24/learning-concept-checking-the-hard-way-2/), we'll implement our own short version of concept
checking. It involves some really cool template tricks and C++ trivia.

Feel free to comment and give suggestions.

References:

1. http://www.boost.org/doc/libs/1_55_0/libs/concept_check/concept_check.htm


