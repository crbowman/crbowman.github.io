---
layout: post
title: D'ya Like DAGs?
description: "An example directed acyclic graph in Clojure."
modified: 2018-5-2
tags: [data structures, graphs, clojure]
image:
  feature: generative-8.png
---
\**Work In Progress*\*
<figure>
    <img src="/images/good-dags-dya-like-dags.jpg" alt="">
	<figcaption>
	  <i>Snatch,</i> 2000
	</figcaption>
</figure>

So, do you like DAGs? No I'm not alking about man's furry best friend. I'm talking about every programmers best friend, the directed acyclic graph. Now, you may be saying to yourself "I don't even know what that is, how could it be my best friend?", but if you keep your code in a version control system like git or mercurial, then you've been using a DAG every day and didn't even know it. So in this post I'll go over a high level overview of DAGs, and show you how to implement one in Clojure. 

## What is a DAG?

So I already told you that DAG stands for a directed acyclic graph, but what does that even mean. A DAG is a graph that only flows in only one direction, and contains no cycles. Like every graph, a DAG is a collection of nodes that are conected by edges, but the edges in a DAG have a direction. So an edge from node A to node B is no the same a an edge from node B to node A. When we say that a DAG contains no cycles (acyclic), it means that no node can be a child of itself. More simply put, if you were to move through the graph from node to node by following the edges, you would never encounter the same node twice. 


