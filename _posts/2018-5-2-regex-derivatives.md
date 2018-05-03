---
layout: post
title: Regular Expression Derivatives
modified: 2018-5-2
tags: [clojure, algorithms, regex]
image:
  feature: generative-2.jpg
---
## Regular Expression Derivatives 

This post is inspired by Matt Might's original regex derivative implementation in scheme[^1].


{% highlight clojure %}
(defn concatenation? [re]
  (= (:op re) :cat))
  
(defn alternation? [re]
  (= (:op re) :alt))

(defn repetition? [re]
  (= (:op re) :rep))

(defn null-string? [re]
  (= (:op re) :null-string))

(defn null-set? [re]
  (nil? re))

(defn atomic? [re]
  (or (char? re) (symbol? re)))
{% endhighlight %}


{% highlight clojure %}
(defn null-string []
  {:op :null-string})

(def null-set
  nil)
{% endhighlight %}


{% highlight clojure %}
(defn cat [e1 e2]
  (cond (or (null-set? e1) (null-set? e2)) null-set
        (null-string? e1) e2
        (null-string? e2) e1
        :else {:op :cat, :e1 e1, :e2 e2}))
{% endhighlight %}


{% highlight clojure %}
(defn alt [e1 e2]
  (cond
   (null-set? e1) e2
   (null-set? e2) e1
   :else {:op :alt, :e1 e1, :e2 e2}))
{% endhighlight %}


{% highlight clojure %}
(defn rep [e]
  (cond
   (null-set? e)   null-string
   (null-string e) null-string
   :else {:op :rep, :e1 e}))
{% endhighlight %}


{% highlight clojure %}
(defn nullable? [re]
  (cond (null-string? re)   null-string
        (null-set? re)      null-set
        (atomic? re)        null-set
        (repetition? re)    null-string
        (concatenation? re) (and (nullable? (:e1 re))
                                 (nullable? (:e2 re)))
        (alternation? re)   (or (nullable? (:e1 re))
                                (nullable? (:e2 re)))
        :else null-set))
{% endhighlight %}


{% highlight clojure %}
(defn regex-derivative [re c]
  (cond (null-string? c)    re
        (null-string? re)   null-string
        (null-set? re)      null-set
        (atomic? re)        (if (= c re) null-string
                                         null-set)
        (concatenation? re) (alt (cat (regex-derivative (:e1 re) c) (:e2 re))
                                 (cat (nullable? (:e1 re)) (regex-derivative (:e2 re) c)))
        (alternation? re)   (alt (regex-derivative (:e1 re) c)
                                 (regex-derivative (:e2 re) c))
        (repetition? re)    (cat (regex-derivative (:e1 re) c)
                                 (rep (:e1 re)))
        :else null-set))
{% endhighlight %}


[^1]: [A Regular Expression Matcher in Scheme Using Derivatives](http://matt.might.net/articles/implementation-of-regular-expression-matching-in-scheme-with-derivatives/)
