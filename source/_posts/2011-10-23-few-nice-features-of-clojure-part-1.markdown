---
layout: post
title: "A Few Nice Features Of Clojure, Part #1: Using Macros To Freeze Time"
date: 2011-10-23T18:25:00-07:00
comments: true
categories:
 - clojure
---

I thought it might be fun to show some of Clojure's features off by taking a look at how I use some of them in my own project [EmailClojMatic](https://github.com/AlexBaranosky/EmailClojMatic). I've got 4 features in mind and will show one feature each post.

**Problem #1:**

We want to unit test a function that depends on the current date.

**Solution #1:**

Fortunately we are using [JodaTime](http://joda-time.sourceforge.net/), a great Java date/time library. JodaTime has built in features for setting the 'now' time. Let's use those to create a function to wrap our test code in, that will set the time at a given DateTime.

``` clojure
(defn do-at* [date-time f]
  (DateTimeUtils/setCurrentMillisFixed (.getMillis date-time))
  (try
    (f)
    (finally (DateTimeUtils/setCurrentMillisSystem))))

;; you might use this code like this:

(do-at* (DateMidnight. 2000 1 1)
  (fn [] (DateMidnight.))) ;;=> (DateMidnight. 2000 1 1)
```

If you are like me and you find yourself writing test code using this construct frequently you can use Clojure's macro functionality to essentially extend the Clojure language to have a new feature for evaluating an expression at a given DateTime. This way we won't have to always wrap the expression in a function like we are doing in the do-at* function.

Let's see the macro version that I decided to call `do-at`, because it is just like the built in Clojure function `do` except it evaluates at the given time. [It's probably worth noting that xyz* function with xyz companion macro is a common Clojure idiom. -@darevay]

``` clojure
(defmacro do-at [date-time & body]
  "like clojure.core.do except evalautes the
  expression at the given time"
  `(do-at* ~date-time
    (fn [] ~@body)))

;; nice. Now we can see the actual code that was hiding
;; behind all of those parens and fn creation:

(do-at (DateMidnight. 2000 1 1)
  (DateMidnight.)) ;;=> (DateMidnight. 2000 1 1)
```

It's pretty powerful the way that macros give you the ability to shape your library so that it is free of boiler plate.
