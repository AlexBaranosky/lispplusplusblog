---
layout: post
title: "A Scala Writer Monad"
date: 2011-10-05T20:00:00-07:00
comments: true
categories:
 - monad
 - scala
---

In  Scala we’re quite familiar with the built in Monads:

*   Option
*   List
*   Either Left Projection
*   Either Right Projection

There are other cool things you can do with the Monad pattern, though, one of
which is termed the  Writer monad.

“This  data type represents logging in a pure functional environment where you  maintain compositionality of your code (i.e. no side-effects) and also,  brevity of code.” - [ http://blog.tmorris.net/the-writer-monad-using-scala-example/](http://blog.tmorris.net/the-writer-monad-using-scala-example/)

## Writer Monad 0.1

``` scala
case class Writer(value: Int, diary: String) {
  def flatMap(f: Int => Writer) = {
      f(value) match {
          case Writer(result, d) => Writer(result, diary + d)
      }
  }
  def map(f: Int => Int) = Writer(f(value), diary)
}
// This enables us to say:
new Writer(2, "").flatMap { new Writer(3, "three") }
                 .flatMap { new Writer(4, "four") }
// => Writer(4, "threefour")
```

## There's a Problem. It’s not generic. Let's make the value type generic.

``` scala
case class Writer[A](value: A, diary: String) = {
   def flatMap[B](f: A => Writer[B]) = {
       f(value) match {
           case Writer(result, d) => Writer(result, diary + d)
       }
   }    
   def map[B](f: A => B]) = Writer(f(value), diary)
}
```

## But it only works with Strings: Introducing the Typeclass pattern

``` scala
case class Writer[A, D](value: A, diary: D)(implicit m: Monoid[D]) {
  def flatMap[B](f: A => Writer2[B, D]) =  f(value) match {
      case Writer(result, d) => Writer(result, m.append(diary, d)
  }
  def map[B](f: A => B) = Writer[B, D](f(value), diary)
}
trait Monoid[T] {
   def zero: T
   def append(a: T, b: T): T
}
object Monoid {
   implicit val StringMonoid = new Monoid[St ring] {
       override def zero = ""
       override def append(s1: String, s2: String) = s1 + s2
   }
} 
```

## Our new version still works

``` scala
Writer(5, "").flatMap { num => Writer(num + 3, "added three")
             .flatMap { Writer(_ * 5, "times five")}

// => Writer(40, "added threetimes 5") 
```

## Why didn’t we just extend String with a trait?

Notice  we could have implemented this using traits, and specified in the type  signature that type D of diary had to extend Monoid. Had we done that,  we would have needed to own the String class (which we don’t have access  to) so it could extend trait Monoid, or we would have had to extend it  manually at run-time.

``` scala
object Monoid {
 implicit val StringMonoid = new Monoid[String] {
     override def zero = ""
     override def append(s1: String, s2: String) = {
             val  delimiter = if (s1 == zero || s2 == zero) "" 
                              else ", "
             s1 + delimiter + s2
     }
 }
}

Writer(5).flatMap { n => Writer(n + 3, “added three”)
           .flatMap { n => Writer(n * 5, "times five") }
           .flatMap { n => Writer(n + 2, "plusTwo") }
// => Writer(42, "added three, times 5, plus two") 
```

## Watch how using the typeclass pattern made it simple to use other classes as the ‘diary’ for our Writer

``` scala
object Monoid {
    implicit val StringBuilderMonoid = new Monoid[StringBuilder] {
        override def zero = new StringBuilder("")
        override def append(sb1: StringBuilder, sb2: StringBuilder) = {
            val  delimiter = if (sb1.toString == "" || 
                                 sb2.toString == "") "" 
                             else ", "
            sb1.append(delimiter).append(sb2)
        }
    }
}

Writer(22, new StringBuilder("")).flatMap { n => 
    Writer(n + 2, new StringBuilder("plus two")) 
}.flatMap { n => 
    Writer(n * 4, new StringBuilder("times four")) 
})

// => Writer(96, StringBuilder("plus two, times four")) 
```

## What we learned

We could get our Writer to work with a 3rd party class without needing to change either the Writer or the StringBuilder code.

This Writer pattern can be used with anything that starts from nothing (zero) and can be built up piece by piece (diary). In essence Writer Monad is a sort of immutable, functional Builder pattern. 

We  saw that we're not limited to the monads Scala comes packaged with. We  can
use our own monads to define computational contexts to suit our  code's unique
needs.

## Writer Monad Materials 

*  [Learn You a Haskell for Great Good](http://learnyouahaskell.com/for-a-few-monads-more#writer)
*  [Scalaz Writer](http://scalaz.github.com/scalaz/scalaz-2.9.0-1-6.0/doc.sxr/scalaz/Writer.scala.html)
