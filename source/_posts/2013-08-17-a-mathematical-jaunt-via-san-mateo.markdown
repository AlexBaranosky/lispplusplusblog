---
layout: post
title: "A Mathematical Jaunt Via San Mateo Parking Meters"
date: 2013-08-17T13:37:00-07:00
comments: true
categories:
 - probability
 - life
 - math
---

Theeee other day some colleagues and I drove into downtown San Mateo for lunch.  An interesting decision had to be made: how many quarters to pay the meter. 

The price for parking is $0.25 per half hour. Your penny-pinching instincts suggest to only put in 2 quarters.  Let's say that a parking ticket for going over the meter is $40.

## Breaking Down the Scenario

Let's apply a little reasoning and mathematics to this, situation and see what's the right amount of quarters to put into the meter.

There are 5 possible outcomes:

1.  pay $0.50, go under one hour, no ticket: total price $0.50
2.  pay $0.50, go over one hour, no ticket: total price $0.50
3.  pay $0.50, go over one hour, get ticket: total price $40.50
4.  pay $0.75, go under one hour, no ticket: total price $0.75
5.  pay $0.75, go over one hour, no ticket: total price $0.75 

## Conservative Guesstimates

Now the fun part.  Let's assign rough guesstimates of likelihood for each outcome.

Let's be generous and say that you only go over one hour 20% of the time, and only 20% of the times you go over do you get a ticket.

Let's also say that you pay $0.50 half the time and $0.75 half the time.

1.  40% probability x $0.50 = $0.20
2.  8% probability x $0.50 = $0.04
3.  2% probability x $40.50 = $0.81
4.  40% probability x $0.75 = $0.30
5.  10% probability x $0.75 = $0.075 

Computed average cost: $1.425

Even at the conservative rate of ticketing of 20% of time you go over an hour, you still increase your expected rate of pay to almost double, by choosing to pay $0.50 half the time!

## With Vigilant Meter Maids

If instead you think you'll get a ticket 75% of the time you go over, the average cost per park becomes a whopping $3.625

1. 40% probability x $0.50 = $0.20
2. 2.5% probability x $0.50 = $0.0125
3. 7.5% probability x $40.50 = $3.0375
4. 40% probability x $0.75 = $0.30
5. 10% probability x $0.75 = $0.075 

Computed average cost: $3.625

So it looks like in general it pays to be safe and pay the extra quarter when parking in downtown San Mateo.

## Further Precision

We could expand this further, by considering likelihoods of lunch going over 1.5 hours.  I didn't because I think there's diminishing returns on investment because the odds of going over 1.5 hours are probably only like 20% of 20% => 4%...  I'll leave that as an exercise for the reader :)
