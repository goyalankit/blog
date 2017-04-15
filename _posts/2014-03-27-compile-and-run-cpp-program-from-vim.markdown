---
layout: post
title: "Compile and run cpp program from Vim"
date: 2014-03-27 22:07
comments: true
categories: vim tip snippet
---

To compile and run cpp program from `MVIM`(or VIM) directly. Add this to
your ~/.vimrc

{% highlight sh %}
nnoremap <C-c> :!g++ -o  %:r.out % -std=c++11<Enter>
nnoremap <C-x> :!./%:r.out
{% endhighlight %}

Note: You can **remove/add** the `<Enter>` if you prefer to press `<Enter>` yourself.

Now simply hit `Ctrl + c` to compile and `Ctrl + x` to execute.


For a source file with name `main.cpp`, commands will be expanded to

{% highlight sh %}
g++ -o main.out main.cpp -std=c++11
./main.out

{% endhighlight %}

The `%:r` gets the current filename without the extension whereas `%`
gets the current filename.

You can very easily extend above approach to compile using **makefile**.


