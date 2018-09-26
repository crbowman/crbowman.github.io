---
layout: post
title: D'ya Like DAGs?
modified: 2018-5-2
tags: [data structures, graphs, clojure]
image:
  feature: generative-8.png
---

<figure>
    <img src="/images/good-dags-dya-like-dags.jpg" alt="">
	<figcaption>
	  <i>Snatch,</i> 2000
	</figcaption>
</figure>

So, do you like DAGs? No I'm not alking about man's furry best friend. I'm talking about every programmers best friend, the directed acyclic graph. Now, you may be saying to yourself "I don't even know what that is, how could it be my best friend?", but if you keep your code in a version control system like git or mercurial, then you've been using a DAG every day and didn't even know it. So in this post I'll go over a high level overview of DAGs, and show you how to implement one in Clojure. 

## What is a DAG?

So I already told you that DAG stands for a directed acyclic graph, but what does that even mean. A DAG is a graph that only flows in only one direction, and contains no cycles. Like every graph, a DAG is a collection of nodes that are conected by edges, but the edges in a DAG have a direction. So an edge from node A to node B is no the same a an edge from node B to node A. When we say that a DAG contains no cycles (acyclic), it means that no node can be a child of itself. More simply put, if you were to move through the graph from node to node by following the edges, you would never encounter the same node twice. 

$$\require{\tikz-cd}$$

$$
\begin{figure}
    \centering
    \begin{tikzcd}[column sep=.8cm, row sep=1.5cm, cells={nodes={draw, circle}}]
        &[-.7cm]&[-.7cm]&1\arrow{dlll}[swap]{18}\arrow{dl}[swap,near end]{12}\arrow{d}[swap]{9}\arrow{dr}[swap]{11}\arrow{drrr}{14}&&[-.7cm]&[-.7cm] \\
        2\arrow{drrr}[swap,near end]{19}\arrow[end anchor=165]{drrrrr}[pos=.05]{16} && 3\arrow{dl}[swap,near end]{23} & 4\arrow{d}[swap,near start]{27}\arrow{drr}[swap,near start]{23} & 5\arrow{dr}[near end]{13} && 6\arrow{dlll}[near start]{15} \\
        & 7\arrow{drr}[swap]{17} & & 8\arrow{d}[swap]{11} & & 9\arrow{dll}{13} & \\
        &&&1&&&
    \end{tikzcd}
    \caption {Directed Acyclic Graph}
\end{figure}
$$
