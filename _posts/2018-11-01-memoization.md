---
layout: post
title: Memoization - Remember what we already found
date: 2018-11-01 12:00
comments: true
external-url:
categories: Python
tags: [python, memoization, algorithmic]
permalink: /blog/:categories/memoization
---

> Certain problems can be solved quite easily using recursion. It is the process in which a function calls itself directly or indirectly by defining a base case. Sometimes, using recursion can be a bit complicated, like if the base case is not reached or not defined, which can be very slow.
<br/>Memoization is a very effective technique that accelerates recursion. So what is memoization ? And how does it work ?


##1. Overview
Let $f$ be a function. Memoizing $f$ is switching between the invocations of $f$ and checking whether the value of an input $x$ can already be found in the memory whenever $f(x)$ is to be computed. If $x$ is not already present in the memory, we compute $f(x)$ using its recursive definition and store the resulting value in it.

Memoization is similar to a very familiar technique we already know in computer science, **caching** ! The aim goal of caching techniques is **avoid doing the same thing repeatedly to avoid spending unnecessary running time or resources**.

To understand how the memoization works and why is it effective, let's take a small and simple example. First, we will introduce a naive solution then try to optimize it using memoization.

##1. Memoization in example
###1. The Fibonacci sequence
Let's take the Fibonacci sequence, which defines every number after the first two as the sum of the two preceding ones. Its recurrence relation is defined as :

$$ \begin{equation} F(n) = F(n-1) + F(n-2) \end{equation} \label{eq:fibo} $$

where seed values are,

$$\begin{cases}
F(0) = 0, \\
F(1) = 1.
\end{cases}
$$

To calculate the Fibonacci sequence defined in \eqref{eq:fibo}, we can use an iterative or a recursive version. Beside the disadvantages of recursion (slow), its main advantage is making the algorithm a little easier and more readable.

```python
# -*- coding: utf-8 -*-

def fibo_iterative(n):
	"""
	Iterative version of Fibonacci sequence
	"""
	assert n >= 0, "shouldn't be a negative number."
	n2, n1 = 0, 1 # seed values are 0 and 1
	for i in range(n):
		n2, n1 = n1, n2 + n1
	return n2

def fibo_recursive(n):
	"""
	Recursive version of Fibonacci sequence
	"""
	assert n >= 0, "shouldn't be a negative number."
	if n < 2: # seed values, if n equals 0 or 1
		return n
	else:
		return fibo_recursive(n-1) + fibo_recursive(n-2)
```

Let's try to run these two functions and compute the running time of $F(50)$,

```python
if __name__ == '__main__':
	n = 50
	print("(recursive version) F({}) = {}".format(n, fibo_recursive(n)))
	print("(iterative version) F({}) = {}".format(n, fibo_iterative(n)))
```

```text
(recursive version) F(50) = 102334155 (time : )
(iterative version) F(50) = 102334155 (time : )
```

<figure><center>
	<img src="{{ site.baseurl }}/assets/images/fibo_tree.png" width="100%" height="100%">
	<figcaption><b>Figure 1 -</b> The Fibonacci sequence.</figcaption>
</center></figure>