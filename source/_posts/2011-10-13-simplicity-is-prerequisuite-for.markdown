---
layout: post
title: "Simplicity is a Prerequisuite for Reliability"
date: 2011-10-13T09:04:00-07:00
comments: true
---

_(This blog post is based on [Rich Hickey's](http://www.codequarterly.com/2011/rich-hickey/) keynote speech at Strange Loop 2011)_

## Word origins
*  Could we go back to what these words REALLY mean?
*  Simple: sim-plex - one fold/braid
*  Complex: many folds/braids
*  Easy: ... lie near

## Simple

*  One fold/braidOne roleOne task!One conceptOne dimension
*  But not one instance one operation it's about lack of interleaving, it's not cardinality
*  _Interleavage is an Objective notion_

## Easy

*  Near, physically on our own hard drive, in our toolset, IDE, apt get, gem install.
*  Near, to our current understand/skill set familiar - mentally near.
*  We're generally overly fixated on these two meanings of easy.
*  If you want everything to be familiar, you learn NOTHING.
*  There's a third meaning of easy:
*  Near, to our capabilities.
*  _Easy is RELATIVE - unlike simple_

## Limits

*  We can only hope to make reliable those things we understand.
*  We can only consider a few things at a time.
*  Intertwined things must be considered together. This is a mental burden and the intertwining tends to grow combinatorially.
*  Complexity undermines understanding.

## Simplicity Driven Development

*  If you ignore complexity - you will invariably sloooowwww down over time.
*  If you focus on ease - you will have a fast at start, and slow down every iteration to redo everything.
*  If you focus on simplicity - you will go a bit slower at the start, but then ramp up to a higher steady state of productivity.** **

## What is In Your Toolkit?

*  Complex 
*  Simple 
*  state, objects, values, methods, functions, namespaces, vars, managed refs,
inheritence, switch, matching polymorphism, syntax data, imperative loops, fold
set functions, actors queues, ORM declarative data manipulation, conditionals,
rules, inconsistency consistency

## Complect 
archaic word meaning "to braid together"
don't do it
complecting things is a source complexity
best to avoid in the first place

## Compose

*  meaning "to place together"
*  composing simple components is the way we write robust software!
*  We can make simple systems by making them modular.
*  What do we want to make these two parts of code allowed to think about??
*  In other words, program towards abstractions!
*  Don't be fooled by code organization. Two things can be physically separate, but to change one means you need to change the other - they are complected

## State is Never Simple

*  It complects value and time
*  But it is so EASY!
*  It interleaves everything that touches it, directly or indirectly.

## Programming Features That Complect

**Feature Complects**

State everything they touch

Objects state, identity, value

Methods function and state, namespaces

Syntax meaning, order

Inheritance  types
switching/matching multiple who what pairsvariables  value, timeloop, fold     what/howactors    what to do, who does itORM OMG
_Simplicity is NOT the same as Easy - Remember that - It is KEY_**
****The Simplicity Recipe**

You do not need C#/Java/C++. You can make big systems with dramatically simpler tools.

We make hundreds of variations that make it tough to manipulate the essence of this stuff - data. We should use simple data so we can write code to work on the essence of this stuff.
Develop sensibilities around entanglement - learn to see the complecting.

Tools for reliability like testing, refactoring, etc are all safety nets that are orthogonal to this problem.

Choose simple constructs over constructs that encourage braided together code.

Simplify the problem space before you start.
Simplicity often mean you have more things... it's not about counting.
Reap the benefits!
