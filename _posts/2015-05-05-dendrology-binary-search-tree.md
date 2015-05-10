---
layout: post
title: Binary Search Trees In Clojure
description: "Creating our own binary search tree in Clojure"
modified: 2015-5-5
tags: [clojure, algorithms, tree, graph, code]
image:
  feature: bw_graph.jpg
---

A binary tree is a set of nodes each containing a value and a pointer to a left and right subtree that are themselves binary search trees. A binary search tree(BST) is a binary tree such that for every node *n* in the tree every value in the left subtree is less than or equal to the value of *n*, and every value in the right subtree is greater than the value of *n*.

To represent a binary search tree node in clojure we are going to define a struct with three keys, **:left**, **:val**, and **:right**.

{% highlight clojure %}
(defstruct binary-node :left :val :right)
{% endhighlight %}

Now that we have a way to create a tree node, we need a way to insert a new node into an existing BST. We're going to define a function called **binary-insert** that has two parameters, the value we want to insert, and the root of the BST we want to insert it into. However, if we don't supply an existing BST we want to create a new one with our value in the root node. 

When we're inserting into an existing tree we need to make sure the 

{% highlight clojure %}
(defn binary-insert
  "Insert val into binary tree at node root, if root is
   not supplied create a new binary tree with one node"
  ([val]
     (struct binary-node nil val nil))
  ([val root]
     (cond
      (not root)
        (struct binary-node nil val nil)
      (< val (root :val))
        (assoc root :left (binary-insert val (root :left)))
      (> val (root :val))
        (assoc root :right (binary-insert val (root :right))))))
{% endhighlight %}

{% highlight clojure %}
(defn build-binary-tree [xs]
  "Builds a binary tree by iterating through xs, inserting
   each item in the tree"
  (def tree nil)
  (loop [i 0]
    (when (< i (count xs))
      (do 
        (def t (binary-insert (nth xs i) tree))
        (def tree t)
        (recur (inc i)))))
  tree)
{% endhighlight %}

{% highlight clojure %}
(defn binary-tree? [node]
  "True if node is the root of a binary tree"
  (cond
    (and (node :left) (node :right))
      (and (> (node :val) ((node :left) :val))
           (< (node :val) ((node :right) :val))
           (binary-tree? (node :left))
           (binary-tree? (node :right)))
    (node :left)
      (and (> (node :val) ((node :left) :val))
           (binary-tree? (node :left)))
    (node :right)  
      (and (< (node :val) ((node :right) :val))
           (binary-tree? (node :right)))
    (node :val)
      true
    :else
      false))
{% endhighlight %}

{% highlight clojure %}
(defn leaf? [node]
  "True if node is a leaf node"
  (and (not (:left node))
       (not (:right node))))
{% endhighlight %}

{% highlight clojure %}
(defn print-binary-tree
  ([node]
     (if node
       (do
         (println "* " (node :val))
         (if (node :left)
           (print-binary-tree (node :left) 1))
         (if (node :right)
           (print-binary-tree (node :right) 1)))))
  ([node depth]
     (if node
       (do
         (let [tabs (clojure.string/join (repeat depth "  "))]
           (println tabs "* " (node :val)))
         ( if (node :left)
           (print-binary-tree (node :left) (inc depth)))
         (if (node :right)
           (print-binary-tree (node :right) (inc depth)))))))

{% endhighlight %}

{% highlight clojure %}
(defn binary-contains? [node val]
  "True if the binary tree at node contains a node 
   with the value val"
  (cond
   (not node)
     nil
   (< val (node :val))
     (binary-contains? (node :left) val)
   (> val (node :val))
     (binary-contains? (node :right) val)
   :else
     true))
{% endhighlight %}

{% highlight clojure %}
(defn binary-find-min [node]
  "returns the node with the smallest value from the
   binary tree at node"
  (cond
   (not node)
     nil
   (node :left)
     (binary-find-min (node :left))
   :else
     node))
{% endhighlight %}

{% highlight clojure %}
(defn binary-find-max [node]
  "returns the node with the greatest value from the 
   binary tree at node"
  (cond
   (not node)
     nil
   (node :right)
     (binary-find-max (node :right))
   :else
     node))
{% endhighlight %}

{% highlight clojure %}
(defn replace-node-in-parent [par node child]
  "removes node from the binary tree by replacing parent par's 
   reference to node with a reference to node's child"
  (cond
    (= ((par :left) :val) (node :val))
      (assoc par :left child)
    (= ((par :right) :val) (node :val))
      (assoc par :right child)))
{% endhighlight %}

{% highlight clojure %}
(defn binary-remove
  "removes the node with the value val from the binary tree
   at node. Ensures that the tree remains a binary tree"
  ([node val]
     (cond
       (not (binary-contains? node (node :val)))
         node
       (< val (node :val))
         (binary-remove (node :left) val node)
       (> val (node :val))
         (binary-remove (node :right) val node)
       :else
         (binary-remove node val nil)))
  ([node val par]
     (cond
       (< val (node :val))
         (binary-remove (node :left) val node)
       (> val (node :val))
         (binary-remove (node :right) val node)
      :else
        (cond
          (and (node :left) (node :right))
            (let [succ (binary-find-min (node :right))]
              (assoc node :val (succ :val))
              (binary-remove (node :right) (succ :val) node))
          (node :left)
            (replace-node-in-parent par node (node :left))
          (node :right)
            (replace-node-in-parent par node (node :right))
          :else
            (replace-node-in-parent par node nil)))))
{% endhighlight %}