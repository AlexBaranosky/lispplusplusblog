---
layout: post
title: "TDD in Midje in a Nutshell"
date: 2012-02-19T17:41:00-08:00
comments: false
categories:
 - midje
 - clojure
---

[Midje](https://github.com/marick/Midje) is a Clojure testing library supporting a top-down mock function style of development, but also equally well supports a bottom-up style.

This is an introductory article, and depending on the response, there will be more articles about some of the intermediate and advanced features.

## Elegant Examples

Midje starts with a simple premise: that tests should be elegant and readable.

When was the last time you saw code samples like this in somebody's blog, gist, or book?

EXAMPLE:
_As you can see the foo function always returns: a-ham-sandwich_

(foo :bar :baz)
;; => :a-ham-sandwich

As developers, I think we write examples like this, because it reads well to us. We're used to this sort of pseudo-code example style, and so we can all tell at a glance what's happening.

This is where Midje enters. The below is executable Clojure Midje code:

``` clojure
(fact "the foo function always returns :a-ham-sandwich"  
  (foo :bar :baz) => :a-ham-sandwich)
```
You can take that code, and run it, now, and if you've defined foo, then you can run `lein midje` or use midje-mode in Emacs to confirm that foo does in fact always return `:a-ham-sandwich`. (Forget for now that this particular fact doesn't very well cover every possible input. Using a generated test input style of testing could be very useful here, but that is for another blog.)

## Describing Code That Doesn't Exist - Yet

Let's try describing some code that doesn't yet exist with these kinds of examples.  I think if we jot down some example cases, it will help us think more clearly about what we're trying to get our code doing.

``` clojure
(fact "doubles odd numbers"   
  (my-func 3) => 6)  

(fact "triples even numbers"   
  (my-func 4) => 12) 
```

We can run these examples now. First thing you'll notice is that we're getting errors; that's of course because we have not yet written any code. Let's add the my-func function.
``` clojure
(defn my-func [n] )
```
Let's be cautious and just try to get one of these facts working first.
``` clojure
(defn my-func [n]
  (* 2 n))
```

This seems to work... but for only the first fact. (There's a lot to be said for how effectively TDD can help you to stay focused on the task at hand.)

So let's get my-func to pass the second fact as well.

``` clojure
(defn my-func [n]
  (if (odd? n)
    (* 2 n)
    (* 3 n))
```

Running those examples again with lein midje or midje mode, we see that they do indeed pass.

But there's one more thing. There is a little bit of duplication left in my-func. Let's take a second, and refactor this a little.

``` clojure
(defn my-func [n]
  (let [multiplier (if (odd? n) 2 3)]
    (* multiplier n))
```

Now that we've revealed a new concept of a multiplier in our code, it looks like we are done.  Our code works as advertised.

## TDD Recap

You just learned:

*   how to use Midje facts as a form of executable examples
*   how to do TDDNice.

I was sneaky just then. Without letting you know what I was doing, I've walked you through the 4 steps of TDD. Decide what you are testing, write a failing test, make that test pass, refactor the code to have a good design.

## TDD Steps:

1.  We chose a new piece of functionality we wanted the code to have.
2.  We wrote a failing test.  We ran it.  We saw it fail. In Midje, we like to think of this step as taking some time to describe the facts of the future version of the code.
3.  We wrote as little code as possible to make the test pass. In Midje, we think of this step as bringing the code up to date with the new facts.
4.  We took some time to reorganize the code - giving it better structure, removing duplication, adjusting the naming, etc.Repeat.

That's TDD in Midje in a nutshell.

What did you think? Interested in hearing some more intermediate or advanced TDD and Midje concepts and exercises?

## Technicalities

``` clojure
;; project.clj  
(defproject my-project "0.1.0"  
  :dev-dependencies [[lein-midje "1.0.8"]
                     [midje "1.4.0-SNAPSHOT"]])   

;; example with ns macro  
(ns my-ns.t-core   
  (:use my-ns.core       
        midje.sweet))  

(fact   
  (foo :bar :baz) => :a-ham-sandwich)
```

<h2>Comments</h2>
<div class='comments'>
<div class='comment'>
<div class='author'>id</div>
<div class='content'>
Look closely at this screenshot (note: with noscript allowing all scripts except for the notorious google tracking bs) : http://bit.ly/IyQf46<br /><br />&quot;As developers, I think we write examples like this (...)&quot;<br /><br />As developers, *we* know that to mark up *code* you don&#39;t use a *script* tag.<br />a) as a noscript user, your reader won&#39;t know which of the dozen bs js sources to (temporarily) allow<br />b) even if you allow js, it won&#39;t work. &quot;Can&#39;t find brush for: clj&quot;<br />c) you can&#39;t just read it. No JS: no content? Fail!<br /><br />For a bit of potential syntax highlighting, you break your article. I applaud you, dear developer.</div>
</div>
<div class='comment'>
<div class='author'>nodename.com</div>
<div class='content'>
Yes, keep it going!</div>
</div>
</div>
