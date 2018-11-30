---
layout: post
title: Binary Search Trees In Clojure
description: "Creating our own binary search tree in Clojure."
modified: 2015-5-5
tags: [clojure, algorithms, tree, graph, code]
image:
  feature: generative-1.jpg
---

## What is a Binary Search Tree?
\**Work In Progress*\*

A binary search tree (BST) is a binary tree that conforms to the following properties:
* The left child subtree of a node contains only nodes with values less than the parent node’s value.
* The right child subtree of a node contains only nodes with values greater than the parent node’s value.
* Both the left and right child subtrees must also be binary search trees.

### Structure

There are a few different ways we can represent our tree nodes in Clojure, such as records, structs, or just a map. Each option gives us a clojure persistant hash-map under the hood, just with varying levels of abstractions on top. For a small project like this, a regular old map should work just fine.


To represent a BST nodes as a clojure map we are going to need to be able to access a node's value, and it's left and right subtrees. So we just need a map that contains three keys, **:val**, **:left**, and **:right**. So a leaf node (a node with no subtrees) with the value 8 would looke like this:

{% highlight clojure %}
{:val 8 :left nil :right nil}
{% endhighlight %}

However, if we don't have a left or right subtree for our node, we don't actually need our map to have **:left** and **:right** keys in our node. If we try and access key in a map that doesn't contain that key, clojure already returns a nil.

{% highlight clojure %}
(:left {:val 8 :left nil :right nil}) -> nil

(:left {:val 8}) -> nil
{% endhighlight %}

It's usually best practice to never have nil values for map keys in Clojure for this reason. It leads to needless overspecification of your data, and makes it more difficult to visually parse your printed data. When we have a parent node that does have subtrees, it will look like this:

{% highlight clojure %}
{:val 8
 :left {:val 4}
 :right {:val 12}}
{% endhighlight %}


### Insertion

Now that we have a way to create a tree node, we need a way to insert a new node into an existing BST. We're going to define a function called **binary-insert** that has two parameters, the value we want to insert, and the root of the BST we want to insert it into. However, if we don't supply an existing BST we want to create a new one with our value in the root node.

When we're inserting into an existing tree we need to find the
correct insertion point for the new element. We do this through a series of recursive calls, inserting the new element into the left subtree if the element is less than or equal to to root value and inserting into the right subtree otherwise.

{% highlight clojure %}
  (defn binary-insert
    "Insert val into binary tree at node root, if root is
     not supplied create a new binary tree with one node."
    ([val]
       (struct binary-node val nil nil))
    ([val root]
       (cond
        (nil? root) (struct binary-node val nil nil)
        (< val (root :val)) (assoc root :left (binary-insert val (root :left)))
        (> val (root :val)) (assoc root :right (binary-insert val (root :right)))
        (= val (root :val)) root)))
{% endhighlight %}

Now that we have a way to insert elements into an existing tree we can use this to create a BST out of a vector of values. We create an empty tree and then loop through our vector of values, inserting each one into our tree.

{% highlight clojure %}
  (defn build-binary-tree
    "Recursively builds a binary tree out of the elements in xs"
    ([xs]
     (if (empty? xs)
       nil
       (build-tree (rest xs) (binary-insert (first xs)))))
    ([xs node]
     (if (empty? xs)
       node
       (build-tree (rest xs) (binary-insert (first xs) node)))))
{% endhighlight %}

### Printing

{% highlight clojure %}
  (defn print-binary-tree
    ([node]
     (do
       (print-binary-tree (node :right) 1)
       (println (node :val))
       (print-binary-tree (node :left) 1)))
    ([node depth]
     (let [padding (clojure.string/join (repeat depth "    "))]
       (if (nil? node)
         (println padding ".")
         (do
           (print-binary-tree (node :right) (inc depth))
           (println padding (node :val))
           (print-binary-tree (node :left) (inc depth)))))))
{% endhighlight %}

{% highlight clojure %}
  (print-binary-tree (build-binary-tree (range 10)))
{% endhighlight %}

{% highlight clojure %}
                                         .
                                     9
                                         .
                                 8
                                     .
                             7
                                 .
                         6
                             .
                     5
                         .
                 4
                     .
             3
                 .
         2
             .
     1
         .
0
     .
{% endhighlight %}

{% highlight clojure %}
  (print-binary-tree (build-binary-tree (shuffle (range 10))))
{% endhighlight %}

{% highlight clojure %}
                 .
           9
                   .
               8
                   .
       7
           .
  6
           .
       5
                       .
                   4
                       .
               3
                   .
           2
                   .
               1
                       .
                   0
                       .
{% endhighlight %}

### Verification

Now lets write a function that allows us to check a tree to make sure it adheres to the requirements of a BST. We're going to call this function *binary-insert?**, we use a question mark in the function name to signal that the funtion return a boolean value.

{% highlight clojure %}
  (defn binary-search-tree? [node]
    "True if the tree satisfies the binary search property."
    (cond
      (and (node :left) (node :right))
        (and (> (node :val) ((node :left) :val))
             (< (node :val) ((node :right) :val))
             (binary-search-tree? (node :left))
             (binary-search-tree? (node :right)))
      (node :left)
        (and (> (node :val) ((node :left) :val))
             (binary-search-tree? (node :left)))
      (node :right)
        (and (< (node :val) ((node :right) :val))
             (binary-search-tree? (node :right)))
      (node :val) true
      :else false))
{% endhighlight %}

### Searching

{% highlight clojure %}
  (defn binary-contains? [node val]
    "True if the binary tree at node contains a node
     with the value val."
    (cond
      (not node) nil
      (< val (node :val)) (binary-contains? (node :left) val)
      (> val (node :val)) (binary-contains? (node :right) val)
      :else true))
{% endhighlight %}

{% highlight clojure %}
  (defn binary-find-min [node]
    "returns the smallest value in the
     binary tree."
    (cond
     (not node) nil
     (node :left) (binary-find-min (node :left))
     :else (node :val)))
{% endhighlight %}

{% highlight clojure %}
  (defn binary-find-max [node]
    "returns the largest value from the
     binary tree."
    (cond
     (not node) nil
     (node :right) (binary-find-max (node :right))
     :else (node :val)))
{% endhighlight %}

{% highlight clojure %}
  (defn binary-nth [node]
  "returns the nth value from the binary search tree."
    nil)
{% endhighlight %}

### Deletion

{% highlight clojure %}
  (defn leaf? [node]
    "True if node is a leaf node."
    (and (nil? (:left node))
         (nil? (:right node))))
{% endhighlight %}

{% highlight clojure %}
  (defn binary-delete-min [node]
    "Delete the minimum value in a binary tree. Returns a vector with the
     value that was deleted, and the new tree without that value."
    (cond
      (nil? (node :left))
        [(node :val) (node :right)]
      (leaf? (node :left))
        [((node :left) :val) {:left nil :val (node :val) :right (node :right)}]
      :else
        (let [[min left] (binary-delete-min (node :left))]
          [min {:left left :val (node :val) :right (node :right)}])))
{% endhighlight %}

{% highlight clojure %}
(defn binary-delete [node val]
  "Delete a value from a binary tree."
  (cond
    (< val (node :val))
    (assoc node :left (binary-delete (node :left) val))
    (> val (node :val))
    (assoc node :right (binary-delete (node :right) val))
    :else
    (cond
      (leaf? (node :right)) (node :left)
      (leaf? (node :left)) (node :right)
      :else (let [[min right] (binary-delete-min (node :right))]
              {:left (node :left) :val min :right right}))))
{% endhighlight %}
