---
layout: post
title: The Benefits of Immutable Data Structures
date: 2014-05-09 15:05:06
comments: true
---

Recently I've been playing around with Clojure. Clojure, as opposed to Ruby, is a functional programming languages. One of the key distinctions between functional and object-oriented languages is immutable data structures. Immutable data structures, as opposed to mutuble structures, cannot be changed once created. When you change the value of an immutable data structure, you are actually creating a new copy of that structure. A string or an array in Ruby is mutable. Let's prove that with a simple example I just cooked up in IRB.

```ruby
> a = "hello"
#=> "hello"

> a.object_id
#=> 70270455113460

> a[2] = "F"
#=> "F"

> a
#=> "heFlo"

> a.object_id
#=> 70270455113460
```

As you can see, we created a new string called "hello" and assigned it to the variable "a". We modified the third letter in the string but the object_id stayed the same. 
It's the same string, mutated.

I should point out that Ruby is actually pretty unique in this regard. In most other languages such as Python and Javascript, strings are immutable. However, Ruby is designed to be mutable. Objects are designed to encapsulate as much logic as possible and hide everything it can. If you want to check whether an object changed from the last time you looked at it, there's no way to know if it's changed besides for inspecting it.

Imagine you are constructing a class that must receive an object in a specific state. You have no way of knowing by default in Ruby whether that object is in the state that you aassume it will be. If you want to be sure of an object's state as it's passed to another class, you have to weave up the entire method chain and break down how each function is mututing your object. That becomes confusing really quickly and makes it difficult to onboard new developers to your project.

To prevent these kinds of errors from hapening, some people implemnt the Builder pattern in Ruby and other object-oriented languages. The Builder pattern sets the state of an object exactly as you want it and then locks it down. You never have to guess whether the object is in the middle of contstruction because it's built to a certain state from the beginning. 

But there's an additional challenge associated with mutuble data structures that is fixed with immutability. To demonstrate this, I'm going to steal an example used by David Nolan at the New York Times. 

[React](http://facebook.github.io/react/) is a Javascript framework created by Facebook that takes a unique approach to rendering out a page. The framework is a completely new way of imagining the view layer in a traditional front-end MVC framework. React creates a virtual DOM and tracks it alongside the DOM of the page. React then simply does a diff on the the two DOM's and makes changes accordingly. Game developers have used this kind of processing to render view layers for years but this is one of the first time that it's been brought to the web. Because of this re-rendering a page using React is much faster compared to something like Backbone.

However, there's a downside to React. Since its written in Javascript which uses mutuble objects, React has to check inside each DOM element to see whether it's changed or not. 

This is where Clojure comes in. There's actually an implementation of Clojure called Clojurescript which compiles down to Javascript. David Nolan created a Clojurescript implementation of React called [Om](https://github.com/swannodette/om). What makes Om so unique is that by leveraging immutable data strucutes, it can be up to twice as fast as React. When Om compares the virtual DOM with the real DOM, it doesn't need to inspect each element and dive into every node to check whether there's been a change; by it's very nature, Clojure will tell you whether it's been changed or not. It's almost like having a tiny flag on top of each element that simply marks whether there has been a change. If there was, Om will update the DOM element. If there wasn't, Om will skip it. This kind of information simply doesn't exist in React so each element has to be inspected on every re-render.

Let's jump back to Ruby. If you had a billion element array, there's no way that we would be able to know if there was a change in that array besides for iterating through the entire structure. In Ruby and other object-oriented languages you constantly have to "check-in" with your objects and manage their state.

I've learned a lot playing with Clojure and am looking forward to learning more about the language. Every programming language has something unique to teach which can be learned from.
