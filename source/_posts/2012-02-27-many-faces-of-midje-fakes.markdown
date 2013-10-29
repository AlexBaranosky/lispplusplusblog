---
layout: post
title: "The Many Faces of Midje Fakes"
date: 2012-02-27T19:21:00-08:00
comments: true
categories:
 - midje
 - clojure
---

The [Midje](https://github.com/marick/Midje) Clojure testing library enables top-down development, and the primary tool it has for enabling it is the fake.

In Midje, you don't have mock objects like you do in object-oriented languages, instead you fake out functions. The simplest way you can fake out functions is by using the provided form within your fact.

## Fakes

``` clojure
(unfinished doubler)

(defn six-times [x]
  (* 3 (doubler x)))

(fact
  (six-times ..n..) => 12
  (provided
    (doubler ..n..) => 4))
```

(Things that look like ..x.. are called Midje metaconstants.  For now just pretend they are constants, or skip to the end of this article for a link to the wiki.)

In the above example we've managed to define six-times in terms of a function, doubler, that is not yet defined (it is unfinished).

Presumably the next thing we'd do when designing a piece of software like this is to write the fact for doubler.

``` clojure
(fact 
  (doubler 2) => 4)

(defn doubler [n] (* 2 n))
```

Or perhaps write a more thorough test:

``` clojure
(tabular
  (fact (doubler n) => result)

  n    result
  -2   -4
  -1   -2
  0    0
  1    2
  2    4
  100  200  )
```

(Tabular is a way to write one fact that runs against multiple input sets.)

## The Myriad Uses of Fakes

Fakes have other uses besides simple stubbing.

They can also be used to verify call counts. Imagine that you want to check that when some data is not valid there are no calls to send-email, but otherwise there will be 1 call sent to each recipient?

We can write this kind of test using the :times n prerequisite option. (where n
is some non-negative integer)

``` clojure
(fact "don't send emails when data is invalid"
  (email-job :alex :brian) => irrelevant
  (provided
    (valid-data?) => false
    (email anything) => nil :times 0 ))

(fact "when data is valid send one email to each recipient"
  (email-job :alex :brian) => irrelevant
  (provided
    (valid-data?) => true
    (email :alex) => nil :times 1
    (email :brian) => nil :times 1 ))

;; The implementation

(unfinished valid-data? email)

(defn email-job [& dudes]
  (when (valid-data?)
    (doseq [d dudes]
      (email d))))
```

## Checkers

Using checkers we can even expand on the idea of what can be checked by a provided.  In the above examples first we used the anything checker to check that the email function was never called with anything. Then we simply checked that the email function was called with :alex and :brian once each.  

In this example we want to check that the query function is called with 10 things that look like query maps. To do that we first need to create our own checker that checks whether the parameter looks like a query. 

``` clojure
(defchecker a-query [actual]
  (and (:target-db actual) (:query-plan actual)))
```

Midje will say that any arg matches this checker if it has a non-false/non-nil :target-db and :query-plan.

Let's confirm query is called 10 times with params that look like queries.

``` clojure
(fact "always sends 10 queries"
  (execute-queries) => irrelevant
  (provided
    (query a-query) => irrelevant :times 10))

;; The implementation

(unfinished query)

(defn execute-queries []
  (dotimes [_ 10]
    (query {:target-db :foo       ;; this will pass because it looks like a-query
            :query-plan :bar})))  ;; but (query {:another "map"}) would not pass
```

## The Fake Chain Gang

If you've ever violated The Law of Demeter in an object-oriented language and then tried to use mock objects with the offending class, you may have experienced a chain of mocks.

A chain of mocks is when you mock a method on a mock object that itself returns another mock object, that has a method on it mocked to *finally* return the value you want.

``` java
when(mockA.call()).thenReturn(mockB);
when(mockB.call()).thenReturn(mockC);
when(mockC.call()).thenReturn(42);
```

In Midje the naive approach might be to write something like:

``` clojure
(fact
  (foo) => 42
  (provided 
    (baz) => ..a.. ;; remember, just pretend they're constants
    (bar ..a..) => ..b..
    (qux ..b..) => 42))

;; The implementation

(unfinished baz bar qux)

(defn foo [] 
  (qux (bar (baz))))
```

Midje supplies a feature called Folded Prerequisites that enable a nice condensed syntax for this, though it is probably a code smell if you are doing this too often.

``` clojure
(fact
  (foo) => 42
  (provided
    (qux (bar (baz))) => 42))
```

## Fakes That Cover Multiple Facts

There are other ways to make fakes in Midje as well, by using the background macro and the against-background macros. These give you the ability to write fakes that cover a group of fakes.  I think I'll talk about them in another blog though time permitting.

## Wiki Links

*  [Metaconstants](https://github.com/marick/Midje/wiki/Metaconstants)
*  [Using Checkers in Fakes](https://github.com/marick/Midje/wiki/Checkers-within-prerequisites)
*  [Folded Prerequisites](https://github.com/marick/Midje/wiki/Folded-prerequisites)
