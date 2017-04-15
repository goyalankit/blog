---
layout: post
title: "Learning concept checking the hard way - P2"
date: 2014-03-24 18:38
comments: true
categories: code programming cpp templates concept-checking
---

In the [previous post](http://goyalankit.com/blog/2014/03/21/learning-concept-checking-the-hard-way-1/), I talked about importance/use case of concept
checking. In this post we'll step by step see how concept checking
works.

Our final goal is to write a `has_less` method, which for any
given type will return true or false based on whether `<` is defined for
that type or not at the **compile time**

Let's first look at some of the c++ concepts which we'll use in the
process.

## 1. decltype
(since c++11)

`decltype` let's you extract type from any expression. It's available in
C++11

For example, In the example below, decltype can be used to get the type
of variable `myType1` and used to define another variable with the same
type.

{% highlight cpp %}
//dec.cpp file
class MyType{
public:
    MyType(){std::cout << "Mytype: constructor called." << std::endl; }
};

int main(int argc, const char * argv[])
{
    MyType myType1;
    decltype(myType1) myType2;
    return 0;
}

>> ./a.out
Mytype: constructor called.
Mytype: constructor called.

{% endhighlight %}


**Takeaway**: we can use **decltype** to get type of a variable.

## 2. declval
(since c++11)

`declval` creates a default constructed type of its argument for use in type checking. It never returns a value. 
It is usually used with `decltype` to determine the type of an
expression. It doesn't
require constructor of the types to exist or match. Line 14 won't
compile because there is no matching constructor.

Note that the compilation error occurs because `NonDefault` type doesn't
have a "default" constructor. If you uncomment the line number 6, the compilation error on line 14 will go away since now compiler will find a matching constructor.


Example, taken from [cpp reference](http://naipc.uchicago.edu/2014/ref/cppreference/en/cpp/utility/declval.html)

{% highlight cpp linenos %}
  struct Default {
      int foo() const {return 1;}
  };
   
  struct NonDefault {
  //    NonDefault(){}
      NonDefault(const NonDefault&) {}
      int foo() const {return 1;}
  };
   
  int main()
  {
      decltype(Default().foo()) n1 = 1; // int n1
  //  decltype(NonDefault().foo()) n2 = n1; // will not compile
      decltype(std::declval<NonDefault>().foo()) n2 = n1; // int n2
      std::cout << "n2 = " << n2 << '\n';
  }
{% endhighlight %}

## 3. constexpr 
(since c++11)

`constexpr` gives you a constant at the compile time instead of runtime
constant that you get using `const` keyword.

{% highlight cpp %}
constexpr int CreateConstant (int a, int b) { return a * b; }
const int arraySize = CreateConstant(2,3);
{% endhighlight %}

`arraySize` will be calculated at the compile time here.

## 4. static_assert
(since c++11)

Performs compile-time assertion checking

{% highlight cpp %}
//will give you a compile time assertion failure error. 
int main(int argc, const char * argv[])
{
    static_assert(1==2, "Assertion failed.");
    return 0;
}
{% endhighlight %}


## 5. Template specialization

Templates in c++ are turing complete and you can do lot of cool stuff
with them. Some trivia about templates:

* **Templates are instantiated at compile time and only if they are used
   somewhere in a program.** However they are checked for syntax.

   Consider the example below, where we have a generic template method(`hello(T a)`) that
   calls a method(`iDontExist`) that is not defined anywhere. However the code
   compiles successfully since we are not calling that method.

   Note the specialization of the template for int type. When compiler
   has more than one functions with the same name, it looks at the type
   and select the best match. In case of `hello<int>(2)`, it selects the
   specialized template over generic so, it instantiates only the
   specialized template which is both syntactically and semantically correct.


{% highlight cpp %}

// This is a valid code and will compile just fine
template <typename T>
void hello(typename T::foo a){
    int b = T::iDontExist(12); //iDontExist doesn't exist.
    std::cout << a << std::endl;
}

// Another possible candidate.
template <typename T>
void hello(T a){
    std::cout << a << std::endl;
}

int main(int argc, const char * argv[]){
    hello<int>(2);
    return 0;
}
{% endhighlight %}


* **Substitution failure is not an error (SFINAE)**

Note in the above program, when `hello<int>` is called compiler first
tries to match it with the `void hello(typename T::foo a)`, however `int` doesn't have a nested type `int::foo`.
This is called a substitution failure, compiler failed to substitute
typename T for int. 

Compiler doesn't throw a compilation error here; it removes this
overload from potential candidate and tries to
find the next possible match. If one or more candidates remain and overload resolution succeeds, 
the invocation is well-formed. Check out more about SFINANE on [wikipedia](http://en.wikipedia.org/wiki/Substitution_failure_is_not_an_error)

* **Given two possible candidates where one is variadic and other is not.
  Compiler always chooses the non-variadic method.** For example, In the
  example below compiler has two candidates for `hello<int>`. Compiler
  chooses the non-variadic one.

{% highlight cpp %}
template <typename T>
// Note the variadic(...) arguments
void hello(...){
    std::cout << "variadic method." << std::endl;
}
// Another possible candidate.
template <typename T>
void hello(T a){
    std::cout << "non-variadic method." << std::endl;
}
int main(int argc, const char * argv[]){
    hello<int>(2);
    return 0;
}
>> ./a.out
non-variadic method.
{% endhighlight %}


---

That's all we need and in the next post we'll write our `has_less` method.
