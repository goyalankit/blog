---
layout: post
title: "modularize_sinatra - modular sinatra app generator"
date: 2013-07-23 21:41
comments: true
categories: sinatra ruby gem labs
---

[**Sinatra**](http://github.com/sinatra/sinatra) is simple, small and fast. 
> Sinatra is a DSL for quickly creating web applications in Ruby with minimal effort. **- sinatra readme**
>

The only downside is that it doesn't offer you the typical [MVC](http://en.wikipedia.org/wiki/Model–view–controller) like [**Rails**](http://github.com/rails/rails). 

Lot of times Rails is an overkill for a simple application and Sinatra seems like a perfect choice. Since using Sinatra, you could write all your code in a single file, and at one time it becomes really hard to manage the code and you may feel the need of porting it to a framework like Rails. However, you don't need all the features that comes with Rails. What you need here is some kind of modularization in your application. For this very purpose I created a gem called [modularize_sinatra](http://github.com/goyalankit/modularize_sinatra)

> **modularize_sinatra** creates a *Rails like* MVC structure without the overhead. 


[`modularize_sinatra`](https://rubygems.org/gems/modularize_sinatra) is available on rubygems. For installation and usage instructions please visit [modularize_sinatra](http://github.com/goyalankit/modularize_sinatra)

## What does it do?
It generates the following directory structure, when generated using `modularize_sinatra myapp -C user` 

{% highlight sh %}
      .
      |-- Gemfile
      |-- Rakefile
      |-- config
      |   `-- environment.rb
      |-- config.ru
      |-- lib
      |   |-- app.rb
      |   |-- controllers
      |   |   `-- user.rb
      |   `-- views
      |       `-- users
      |           `-- index.erb
      |-- myapp.rb
      |-- public
      |-- script
      |-- spec
      |   |-- controllers
      |   |   `-- user_spec.rb
      |   |-- spec_helper.rb
      |   `-- support
      `-- tmp
  {% endhighlight %}


It generated some usual ruby files like:

* **Gemfile** -  A format for describing gem dependencies for Ruby programs.
* **Rakefile** - Contains task to run specs.
* **config.ru** - Rack configuration file.
* **config/environment.rb** - Here you could load your configuration files. ***Hint**: Could load configuration given in .yml files*
* **lib** contains directories for placing your controllers and views. 
* **myapp.rb** loads all ruby files inside `lib`, `lib/controllers` and `lib/models`.

Note the **lib/models** in above point. You could create a directory called `models` in lib directory to place your models and it will also be loaded.

* **public** - To serve public assets.
* **script** - You could get rid of this directory. This is just a container to put your scripts.
* **spec** - Place your specifications inside this. modularize_sinatra integrates rspec for you by default. 

Please contribute at [modularize_sinatra](http://github.com/goyalankit/modularize_sinatra)

Comments and suggestions are most welcome.
