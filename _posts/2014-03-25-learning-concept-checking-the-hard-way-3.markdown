---
layout: post
title: "learning concept checking the hard way - P3"
date: 2014-03-25 19:19
comments: true
categories: code programming cpp templates concept-checking
---

In the previous posts [this](http://goyalankit.com/blog/2014/03/21/learning-concept-checking-the-hard-way-1/) and [this](http://goyalankit.com/blog/2014/03/24/learning-concept-checking-the-hard-way-2/), I talked about the basics of
concept checking and stuff you need to know.

## Finally here's our `has_less` method.

{% highlight cpp %}
#include <iostream>
#include <vector>
#import <complex>

using namespace std;

template <typename T, typename unused = decltype(T() < T())>
std::true_type has_less_helper(const T&);

template <typename T>
std::false_type has_less_helper(...);

template <typename T>
constexpr bool has_less(void){
    using my_type = decltype(has_less_helper<T>(std::declval<T>()));
    return my_type::value;
}

template <typename T>
class BinarySearchTree{
public:
    static_assert(
        has_less<T>(), 
        "Assertion failed. No less than operation defined."
    );
    BinarySearchTree() {
    }
};

int main(int argc, const char * argv[])
{
    BinarySearchTree<int> binarySearchTree;
    // Compile time error here, since complex<int> doesn't
    // have a '<' operator.
    BinarySearchTree<complex<int>> compBinarySearchTree;
    return 0;
}
{% endhighlight %}

## Let's dissect the above method and see what's happening.


`has_helper` method has two possible overload options. For the `int`
datatype both are possible overload candidates but as we saw earlier
compiler prefers the **non-variadi**c option. So it choosses the first
option which has a return type of `std::true_type`.

However for `complex<int>`, compiler has only one option since `<`
doesn't exist for `complex` type.

{% highlight cpp %}
// we don't need to define the methods
// Since we are not calling them.
// we are just using the principle of SFINAE that applies to template
// arguments and not to the body

template <typename T, typename unused = decltype(T() < T())>
std::true_type has_less_helper(const T&);

//and

template <typename T>
std::false_type has_less_helper(...);
{% endhighlight %}


*Note:* [`true_type`](http://www.cplusplus.com/reference/type_traits/true_type/) and [`false_type`](http://www.cplusplus.com/reference/type_traits/true_type/) are just structs with values as
`true` and `false` resp. Check out [their usage](http://www.cplusplus.com/reference/type_traits/integral_constant/)



We could directly use `has_less_helper` to know what Type has a less
than operator defined for it. However it is important to note that the
`has_less_helper` assumes that type `T` has a default constructor
defined for it(`decltype(T() < T())`).

To remove above dependency we could use declvalue(as discussed in
previous post). That is what we are
doing in `has_less` method.

{% highlight cpp %}
template <typename T>
constexpr bool has_less(void){
    // using is equivalent to typedef
    using my_type = decltype(has_less_helper<T>(std::declval<T>()));
    // note that the return type is either true_type or false_type
    return my_type::value;
}
{% endhighlight %}


*Note:* the `constexpr` keyword makes the method evaluate at compile time. So we will have a true or false value from this method at compile time.

Inside our definition of generic `BinarySearchTree` we put a
`static_assert`(compile time assert) to check that we have a less that
operator define for the type T. 

{% highlight cpp %}
template <typename T>
class BinarySearchTree{
public:
    static_assert(
        has_less<T>(), 
        "Assertion failed. No less than operation defined."
    );
    BinarySearchTree() {
    }
};
{% endhighlight %}

That's it! Now when you declare something like 

{% highlight cpp %}
BinarySearchTree< complex<int> > compBinarySearchTree;
{% endhighlight %}

**We get a compile time error.**

Awesome! Now you understand the concept of concept checking. Happy
coding.
