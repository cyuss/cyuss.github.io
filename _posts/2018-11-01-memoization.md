---
layout: post
title: Memoization - Remember what we already found
date: 2018-11-01 12:00
comments: true
external-url:
categories: Python
tags: [python, memoization, optimization, algorithmic]
permalink: /blog/:categories/memoization
---

> Certain problems can be solved quite easily using recursion. It is the process in which a function calls itself directly or indirectly by defining a base case. Sometimes, using recursion can be a bit complicated, like if the base case is not reached or not defined, which can be very slow.
<br/>Memoization is a very effective technique that accelerates recursion. So what is memoization ? And how does it work ?


##1. Overview
Let $ƒ$ be a function. Memoizing $ƒ$ is switching between the invocations of $ƒ$ and checking whether the value of an input $x$ can already be found in the memory whenever $ƒ(x)$ is to be computed. If $x$ is not already present in the memory, we compute $f(x)$ using its recursive definition and store the resulting value in the memory.