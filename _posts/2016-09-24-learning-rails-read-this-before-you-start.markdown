---
title:  "Learning Rails ? Read This Before You Start"
date:   2016-09-24
categories: [jekyll]
tags: [ruby, "ruby on rails", "ruby metaprogramming"]
---

If you're already a web programmer thinking about learning Ruby on Rails don't learn the hard way like I did by diving straight into Rails. Spend a few hours on some Ruby basics and you'll be up and running on Rails in no time.

1. [Learn some Ruby First](#ruby-first)
1. [Some Ruby](#ruby)
1. [Ruby Metaprogramming](#metaprogramming)
1. [Ruby Gems](#gems)
1. [Ruby on Rails](#rails)


There are many useful step-by-step, getting started guides to Ruby on Rails and I've included some links to the ones I like the most below. Ruby on Rails is a surprisingly big subject and there are fewer explanations of what you are learning and how these parts fit together so this post attempts to address this. 

<a id="ruby-first" ></a>

## Spend an hour or two learning a little Ruby first

If you are new to Ruby on Rails I can't emphasise enough how important it is to understand the difference between Ruby and Ruby on Rails. I know this because I learned Rails first and wasted a lot of time struggling with incorrect assumptions about what it was doing.

One of the first code bases I worked on had this line of code embedded into a web page: 

    <%= link_to 'products', products_path %>
    
It was a web page so the meaning of `link_to` was pretty easy to guess but there was this method `products_path` that returned the string '/products'. I assumed it was a method in the application code base but I couldn't find it anywhere. It seemed unlikely that Rails would have such a domain specific method but I searched the documentation and the source code just in case. It wasn't there either. So what was `products_path` and why on earth was it working ?
 
Had I learned a little Ruby first - especially Ruby meta-programming - the penny would have dropped quite quickly. `products_path` was just one example of Ruby's real beauty as an elegant and expressive programming language; metaprogramming. Metaprogramming allows the language to be adapted to specific domains like web or testing but a little more of that later. 

<a id="ruby" ></a>

## Ruby   

Ruby is a scripted programming language not so different from Python or Perl.

It's dynamically typed, you can change the type of a variable:

    x = 'hello world'
    x = 55
    
But unlike most scripted languages it's also strongly typed you can't mix different objects together:

    x = 'Hello World'
    y = 55
    
    x + y
    TypeError: no implicit conversion of Fixnum into String

    
It has classes:

    class Person
    end
    
    steve = Person.new
   
NOTE: if you're familiar with Java or C# then Ruby classes have a base class of Object in much the same way.
    
It has initialisers:

    class Person
    
      def initialize(name)
        @name = name
      end
      
    end
    
    steve = Person.new('steve')
    
It has class & instance variables and methods

    class Something
 
      #class variables have to be explicitly initialized
      @@some_class_variable = 'I'm shared across all the Something instances'   

      #instance variables can be referenced on the fly  
      def some_instance_method
        @some_instance_variable
      end
      
      def self.some_class_method
        @@some_class_variable
      end
      
    end
    
    
    
    
It has modules and namespaces

    module Human
      class Person
      
        def initialize(name)
          @name = name
        end
        
      end
    end
    
    steve = Human::Person.new('steve')
    
Code blocks can be surrounded by {} or `do end`.

Typically we use {} in a single line and do end in multi line code:

    people.each{ |person| puts person.name }
    
    people.each do |person|
      puts person.name
    end
    
Everything is an object - even nil.

    x.nil? #the NilClass returns true if x is nil
  
### How do I install it ?
    
If you have a Mac then ruby will already be on your machine. At the time of writing the latest stable version is 2.3.1. In the command line run `ruby -v` to find the version you have. 

If your version is a little old **don't try and install a new one** over it. Use a tool like [rbenv](http://rbenv.org) that enables you to download and install multiple versions of ruby on the same machine. For getting started ruby 1.9 or greater should be more than sufficient. 

Try this [20 minute introduction to Ruby](https://www.ruby-lang.org/en/documentation/quickstart/) from the [official ruby website](https://www.ruby-lang.org).

<a id="metaprogramming" ></a>

## Ruby Meta-programming

When you're starting out it's not important to understand the details of ruby metaprogramming but it is important to know that it exists. Rails uses it a lot.

### Extending Classes

Strings in Rails have these handy methods:

    x = 'dog'
    x.pluralize
    => 'dogs'
    
    y = 'dogs'
    y.singularize
    => 'dog'
    
But try either of these is Ruby and you'll get something like:

    undefined method `pluralize' for "dog":String
    
Rails has extended the basic Ruby string class with a method commonly used within its web framework. Rather than creating some clumsy `Pluralizing` class it's just elegantly enabled Rails strings to express the singular and plural versions of each other in a way that seems natural and intuitive to us humans (though maybe not to some Java programmers).

### Method Missing

Rails comes with an Object Relational Bridge called ActiveRecord. ActiveRecord uses metaprogramming tp convert code to SQL and database results to code, again in a nice intuitive way.

    class User < ActiveRecord::Base
    end
    

Our database has a table created by ActiveRecord called 'users' (note the singular `class User` to the plural table `users`). The table has two columns, 'full_name' and 'email'.

Out of the box ActiveRecord allows us to do things like this:

    user = User.create(full_name: 'steve creedon', email: 'steve@creedon.me')
    user.full_name
    => 'steve creedon'
    user.email
    => 'steve@creedon.me'
    
    #this is a typical way to find an object...
    User.where(name: 'steve@creedon.me')

Our user class has no getter or setter methods for `full_name` or `email` but ActiveRecord has utilised another Ruby metaprogramming technique `method_missing`:

Remember that Ruby is like Java and C# in that the Object class is the implicit base class of all classes in Ruby ? Normally when a method is missing Ruby invokes method_missing on this base class which raises an `undefined method` error. 

In the User class above ActiveRecord::Base has overriden `method_missing` so when I call `user.full_name` this invokes `method_missing` in `ActiveRecord::Base`  and ActiveRecord then looks for the data column `full_name`. Of course if I'd written `user.xyz` and there was no column `xyz` in the table then ActiveRecord would simply pass this `method_missing` to the overridden method_missing and we would see something like `undefined method 'xyz' for #<User:0x007fde3254f598>`.


### Monkey Patching

This alters the functionality of existing Ruby methods. Generally this is a bad thing that doesn't enhance Ruby but changes it in a way hat could be very confusing for another programmer using your code. It's used - rarely - to patch bugs in 3rd party Ruby Gems (see below). I say rarely because there are better ways of doing that too.

     
<a id="gems" ></a>

## Ruby Gems

A gem is a package for handling everyday things like database drivers, image uploads, authentication and pretty much anything else you can imagine. You can write your own if you want to share code between your own applications. More commonly we install gems from [rubygems.org](https://rubygems.org/gems).

Ruby comes with a package manager command `gem`. To install the devise gem:

    gem install devise  

The most commonly used gems include:

1. Connector gems for databases including MySql, SqlLite, SqlServer, Oracle, DB2, CouchDB and MongoDB, Neo4J.
1. Paperclip manages uploading of file attachments and connects with ImageMagik to resize uplaoded images.
1. Sidekiq will make method calls almost seamlessly asynchrous so that heavier tasks are managed in the background and webserver responses return quickly.
1. Will Paginate handles finding and displaying of paginated results.
1. Bluecloth and Redcloth handle Markdown so that text input can be converted to/from HTML.
1. Log4R, logging
1. Nokogiri parses HTML used for simple web scraping.
1. Devise manages user authentication
1. Rspec for very expressive testing of code
1. Factory Girl for creating mock testing objects
1. Capistrano for auto-deployment of code and management of applications on remote servers

Most gems have their source code on github. It's always worth spending time reading the source code of commonly used gems (rubyforge) as they will be good ruby code.

When you are considering using a gem:

1. Note how many other people have installed it
2. Note on github the last time it was updated. There are many gems that are no longer supported by their owners. You could, of course, fork that reository and continue the gem yourself.

So I mentioned that Ruby comes with its pacake manager command `gem`. There is a problem with this method of managing gems. If you have multiple applications on your machine that use different versions of the same gem life can start to become complicated. To solve this the bundler gem was created:

### Bundler

Bundler is a gem to manage the gems in a specific application. Every application will have a Gemfile and running `bundle install` will install gems for that application in such a way it won't effect other applications.

A typical application Gemfile looks like this:

    source 'https://rubygems.org'
  
    ruby "2.2.2"
  
    gem 'rails', '4.2.1'
    gem 'sprockets-rails', '~> 2.0' #version 3 is breaking handlebars_assets
  
    gem 'mysql2'
    gem 'unicorn', '~> 4.9.0'
    gem 'thin'
    gem 'whenever'

It really makes managing gems much much easier so I suggest you take a look at [the bundler website](http://bundler.io) 

<a id="rails" ></a>

## Ruby on Rails

There are so many good articles on getting started with Rails that's there is little point me going into too much detail here so I would start with [the official getting started guide](http://guides.rubyonrails.org/getting_started.html).

Specific things to try and learn are:

Rails [routing and named routes](http://guides.rubyonrails.org/routing.html)

[ActiveRecord](http://guides.rubyonrails.org/active_record_querying.html) the Object Relation Bridge

Rspec is the most commonly used testing framework for Rails. Writing tests is a great way to learn Rails coding. 

One thing to bear in mind is 'The Rails Way'. It's an excellent book on Rails but too many people have turned it into a Bible. Rails is an excellent framework for most web applications but if you build one giant Rails app, after a while it's going to start hurting. Rules are for the guidance of wise men and the obedience of fools. 



