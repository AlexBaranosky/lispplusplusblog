---
layout: post
title: "The Evolution of a Macro"
date: 2013-07-28T02:59:00-07:00
comments: true
categories:
 - clojure
 - testing
 - macro
---


At Runa, where I work, I work on a very large Clojure application which we service online merchants with in order to deliver real-time promotions to the browser. The runtime promotion determination core of the application is quite intricate and has both stateful and immutable parts. If not tested properly, you can make the mistake of letting any setup mutable state effect tests other than the intended test case.

I thought it'd be fun to walk you through how I developed a macro to handle this type of situation at work. I think it is a pretty interesting example of how macros can evolve and what considerations might go into how you architect a meaty macro.

_NOTE: The code has been simplified from the production code to better emphasize the important points._

### First Stab At a Stateful Test Macro

Our first stab at this problem solves the basic initial version of the problem. Often this would be all you need to get the job done. This approach enables us to evaluate our test assertion from within a stateful scope, and we use the finally block of the try-catch to ensure that the rules are always set back to their original state when the test is done.

``` clojure
(defmacro with-rules [rules & body]
  `(let [old-rules# @rules]
     (try
       (reset! rules (evaluate-rules ~rules))
       ~@body
       (finally
         (reset! rules old-rules#)))))

(deftest test-priority-level
 (with-rules (def-rule priority-level 5)
  (is (= true (priority-level 5)))))
```

Then if we have some other sort of state, for example machine learning models, we can write another `with-models` macro and combine the macros in a test.

``` clojure
(defmacro with-models [models & body]
 `(let [old-models# @models]
  (try
   (reset! models ~models)
   ~@body
   (finally
    (reset! models old-models#)))))

(deftest test-model-and-level
 (with-rules (def-rule priority-level 5)
  (with-models {:a [1 2 3 4 5 6]
                :b [7 8 9 10 11 12]} ;; insert fancy math stuff here ;)
   (is (= true (found-promotion? (legal-customer-session)))))))
```

Before I make a serious change to the approach we use for this kind of macro, let's remove the bloat from these macros and make them lean mean macros with beefier functions to go with them.

``` clojure
(defn with-rules* [rules body-fn]
 (let [old-rules @rules]
  (try
   (reset! rules (evaluate-rules rules))
   (body-fn)
   (finally
    (reset! rules old-rules)))))

(defmacro with-rules [rules & body]
 `(with-rules* ~rules (fn [] ~@body)))

(defn with-models* [models body-fn]
 (let [old-models @models]
  (try
   (reset! models models)
   (body-fn)
   (finally
    (reset! models old-models)))))

(defmacro with-models [models & body]
 `(with-models* ~models (fn [] ~@body)))
```

Why? Because macros are harder to read, and not composable.

This way of approaching the stateful setup and teardown in our tests gets the
job done, but has a couple issues. As soon as we have another source of state we
need another with-x macro, and when we combine these together we end up over
time with code that looks like this.

``` clojure
(deftest test-motherload
 (with-x X
  (with-y Y
   (with-z Z
    (with-a A
     (with-b B
       (is (= 42 (evaluate)))))))))
```

What I really want here is one macro not 5! I'd like something I can easily extend as new forms of state come up.

### Composing Functions For A More Extensible Approach

What if we could write test-motherload like this:

``` clojure
(deftest test-motherload
 (with-runtime-state {:x X :y Y :z Z :a A :b B}
  (is (= 42 (evaluate)))))
```

And ideally if a C type of state comes up down the line, it'll be straightforward to extend this macro to accomodate it without pulling your hair out dealing with #'s, ~'s, @'s, ''s and `'s.

With a minor change to with-rules* and with-models* we can enable them to be used to compose 0-arity functions into other 0-arity functions.

```clojure
(defn rules-setup-and-teardown [rules body-fn]
 (fn []
  (let [old-rules @rules]
   (try
    (reset! rules (evaluate-rules rules))
    (body-fn)
    (finally
     (reset! rules old-rules))))))

(defn models-setup-and-teardown [models body-fn]
 (fn []
  (let [old-models @models]
   (try
    (reset! models models)
    (body-fn)
    (finally
      (reset! models old-models)))))
```

We can take the testcase evaluation function and transform it into a function that when called will wrap the evaluate-test-case function in setup and teardown of the rules state.

``` clojure
(rules-setup-and-teardown 
  '(def-rule priority-level 5) 
  (fn [] (evalute-test-case)))
```

And we can chain these function compositions together, to incorporate more varieties of setup and teardown as necessary. Note we don't need no fancy macros for this (yet).

``` clojure
(deftest test-model-and-level
 (let [testcase-fn (fn []
                     (is (= true (found-promotion? 
                                   (legal-customer-session)))))
       wrapped-testcase-fn 
         (->> testcase-fn
              (rules-setup-and-teardown '(def-rule priority-level 5))
              (models-setup-and-teardown {:a [1 2 3 4 5 6]
                                          :b [7 8 9 10 11 12]}))]
   (wrapped-testcase-fn)))
```

Remember what we're aiming for is something more like this:

``` clojure 
(deftest test-motherload
 (with-runtime-state {:x X :y Y :z Z :a A :b B}
   (is (= 42 (evaluate)))))
```

This is going to require us to convert the state map, {:x X :y Y :z Z :a A :b B}, into the appropriate setup-and-teardown wrapper functions, and do the wrapping of the body of the macro as well.

The first definition option->setup-and-teardown-fn, maps keys from the state map to its corresponding setup-and-teardown function.

``` clojure
(def ^:private option->setup-and-teardown-fn
 "Add new setup and teardown wrappers here, and the `with-runtime-state`
  macro automatically gets a new functionality"
 {:rules #'rules-setup-and-teardown
  :models #'models-setup-and-teardown})
```

Finally the macro that makes it all fit together. If you look closely, it macro
expands into something very similar to the composed function solution above.

``` clojure 
(defmacro with-runtime-state [options & body]
 (let [setup-and-teardowns
    ;; 1. map each key of the state-map to its a setup-and-teardown fn
    (for [[k state-value] options]
     `((fn [f#]
       (let [setup-and-teardown# (get ~option->setup-and-teardown-fn ~k)]
        (setup-and-teardown# ~state-value f#)))))]
  ;; 2. thread the testcase body fn through each wrapper fn, and eval it
  `((-> (fn [] ~@body)
     ~@setup-and-teardowns))))
```

We can make the macro more user friendly with a couple of asserts on the args passed to it. These kinds of assertions are especially nice in macros, because they help make the macro's unique syntax easier to use for your teammates without them pulling their hair out.

Also, we can be improve it one step further. We'll use clojure.core/partial to create the 1-arity setup-and-teardown functions, rather than wrapping each of them within the for expression.

``` clojure
(def ^:private runtime-test-option-set (set (keys option->setup-and-teardown-fn)))

(defmacro with-runtime-state [options & body]
 (assert (map? options)
     "Options should be a map.")
 (assert (every? runtime-test-option-set (keys options))
     "Options should have :rules and/or :feature-bits keys")
 (let [setup-and-teardowns
    (for [[k state-value] options]
     `((partial (get ~option->setup-and-teardown-fn ~k) ~state-value)))]
  `((-> (fn [] ~@body)
     ~@setup-and-teardowns))))
```

### Refactoring Addendum

I thought a little more about this and realized that even this macro is doing
too much.  We can actually extract the code that wraps a 0-arity function f in
setup-and-teardown functions. In the process we end up with a macro with one
line of code that does the heavy lifting and a few more lines of asserts.  

``` clojure
(defn wrap-in-setups-and-tear-downs [options f]
  (let [setup-and-teardown-fns (for [[k state-value] options]
                                 (partial (get option->setup-and-teardown-fn k) 
                                          state-value))]
    (reduce (fn [f' setup-and-teardown-fn]
              (setup-and-teardown-fn f'))
            f
            setup-and-teardown-fns)))

(defmacro with-runtime-state [options & body]
  (assert (map? options)
          "Options should be a map.")
  (assert (every? runtime-test-option-set (keys options))
          "Options should have :rules and/or :feature-bits keys")
  `((wrap-in-setups-and-tear-downs ~options (fn [] ~@body))))
```

This is a useful step because now we have another higher order function with which to build useful state-management tooling, and we've eliminated more of the potentially confusing macro code. 

### Takeaways

So why go to all this trouble?

*   We've written a macro that is extensible.
*   We've minimized actual lines of macro code. Yay!
*   We get validation of the state-map keys for free, anytime we extend this with a new setup-and-teardown function.
*   Our tests have a clean, uniform, minimal DSL for expressing pre-requisite mutable state.
How did we do it?

*   We took an essentially functional compositional approach.
*   We wrapped it in a layer of syntactic sugar.
Hope you enjoyed this little code walk through.

Comments or questions?

## Comments

<div class='comments'>
<div class='comment'>
<div class='author'>Petr</div>
<div class='content'>
One more reason that using dynamic scope sucks.

BTW Have you thought of replacing set of *-setup-and-teardown with set of
{:setup (fn [] ....) :teardown (fn [] ....)} 
and not repeating (try ... (finally ...)) every time?</div>
</div>
<div class='comment'>
<div class='author'>Alex Baranosky</div>
<div class='content'>
Thanks.  I have at least got it into a stable state that should no longer break in Chrome.</div>
</div>
<div class='comment'>
<div class='author'>regeya</div>
<div class='content'>
cannot find brush for: clj

That&#39;s the dialog that keeps popping up every time I visit a page on this blog.</div>
</div>
<div class='comment'>
<div class='author'>Alex Baranosky</div>
<div class='content'>
Hi Alex, 

Thanks for the comment.  I actually thought I had added the appropriate tags last evening when I wrote it, but must have not saved my change.</div>
</div>
<div class='comment'>
<div class='author'>Alex Ott</div>
<div class='content'>
Hi 

thank you for posting this!
can you also add the clojure label to it, so it will be propagated into Planet Clojure?</div>
</div>
</div>
