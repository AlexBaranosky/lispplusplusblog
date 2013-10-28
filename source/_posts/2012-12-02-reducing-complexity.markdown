---
layout: post
title: "Reducing Complexity"
date: 2012-12-02T02:33:00-08:00
comments: false
categories:
 - clojure
---

I had written some complicated looking code at work recently (shhh, let's keep that between us ok?). Something about it didn't seem right to me, but I was busy at the time so I wrote a TODO to come back and find a way to make it simpler.

Since the code is so confusing it is virtually impossible to read at a glance and understand its intent, I will explain what it does.  It takes in a map where the keys are numbers of days over a given SLA, and the values are the number of shipments that were over the SLA by that amount of days.  The map actually can and does have more keys on it than that, but for the purposes of this article we can ignore them.  The function takes that map and converts it to a version where the values are not counts of the individual SLA, but instead the current running percentage totals.

### An example: 

``` clojure
(cumulative-percentage-past-SLA-map {:past-SLA-1 2 :past-SLA-2 1 :past-SLA-3 3} 4 10)   
;; => {:past-SLA-1 0.60 :past-SLA-2 0.70 :past-SLA-3 100.0}
```

What makes this code end up so complicated is that it is complecting three things: generating the series of sums, dividing them, and creating a new map to return.

``` clojure
(def ^:private all-past-SLA-x-keys [:past-SLA-1 :past-SLA-2 :past-SLA-3])

(defn- cumulative-percentage-past-SLA-map
  "Takes a count-based report and generates percentages that accumulate."
  [count-based-past-SLA-map met-SLA total-shipments]
  (-> (reduce (fn [[past-SLA-map total-past-SLA-so-far] days-past-key]
                (let [updated-past-SLA-map (update-in past-SLA-map
                                                      [days-past-key]
                                                      #(/ (+ total-past-SLA-so-far %) total-shipments))
                      updated-total-past-SLA (+ total-past-SLA-so-far (get past-SLA-map days-past-key))]
                  [updated-past-SLA-map updated-total-past-SLA]))
              [count-based-past-SLA-map met-SLA]
              all-past-SLA-x-keys)
      first))
```

## The solution is to break those 3 parts up.

First we generate the series of sums using [reductions](http://clojuredocs.org/clojure_core/clojure.core/reductions).

Second, we divide each by the total to convert them to percentages.

Thirdly, we create a new map with percents as the values instead of counts.

``` clojure
(defn- cumulative-percentage-past-SLA-map
  "Takes a count-based report and generates percentages that accumulate."
  [count-based-past-SLA-map num-met-SLA total-shipments]
  (let [counts (map #(get count-based-past-SLA-map %) all-past-SLA-x-keys)
        running-totals (rest (reductions + num-met-SLA counts))
        percents (map #(/ % total-shipments) running-totals)]
    (zipmap all-past-SLA-x-keys
            percents)))
```

Another thing you might notice in this code is that I purposely used the more verbose `#(get count-based-past-SLA-map %)` when I could have just used `count-based-past-SLA-map`, as it is already a function!  I prefer to err on the side of making my code as obvious as possible, so I try to avoid using data structures as functions except under a few special cases. But I guess that would be a different topic for a different blog.

<h2>Comments</h2>
<div class='comments'>
<div class='comment'>
<div class='author'>Alex Baranosky</div>
<div class='content'>
I would probably do it that way if I wrote it today, too, without the `get`.  But I don&#39;t think it is a huge deal either way.</div>
</div>
<div class='comment'>
<div class='author'>Timothy Baldridge</div>
<div class='content'>
+1 I&#39;d rather treat functions as maps, than introduce syntactic sugar with #() and %.</div>
</div>
<div class='comment'>
<div class='author'>Alex Baranosky</div>
<div class='content'>
IMO this is an area where Clojure idioms venture into &quot;fancy&quot; territory.  It is a little Perlish.  At the end of the day it is a stylistic concern  and that kind of stuff is something that depends on what your team is most comfortable with. YMMV.</div>
</div>
<div class='comment'>
<div class='author'>Unknown</div>
<div class='content'>
I don&#39;t agree with the last part... In clojure you just don&#39;t use get, it is more idiomatic the other way. Using get just make it more confusing IMHO</div>
</div>
</div>
