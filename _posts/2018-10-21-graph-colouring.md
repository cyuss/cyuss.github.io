---
layout: post
title: Graph Coloring & Chromatic Number
date: 2018-10-21 12:00
comments: true
external-url:
categories: Mathematics
---


> One of the most well known problems in graph theory is the four color theorem for map's coloration. A simple example to understand that problem is, given a geographical map, how many colors are required to color it so that no two adjacent regions have the same color ? The answer is four, but it took a long time to prove this theorem after many false proofs and counterexamples.

<!-- ![useful image]({{ site.url }}/assets/world_map.png){:height="50%" width="50%"} -->
Let $$G$$ be a graph and $$\lambda \in \Bbb{N}^* $$. <br/>We call a proper $$\lambda$$-coloration, a function $$ƒ : V(G) \rightarrow \{1, 2, \dots, \lambda\}$$, where for each $$u, v \in V(G)$$ we have $$ƒ(u) \neq ƒ(v)$$ $$\forall u, v$$ two adjacent vertices of $$V(G)$$. We say that two $$\lambda$$-colorations $$ƒ$$ and $$g$$ are distincts, if for some vertices $$x$$ of $$G$$, $$ƒ(x) \neq g(x)$$.