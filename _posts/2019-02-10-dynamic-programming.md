---
layout: post
title: Dynamic Programming, Solve It Once
date: 2019-02-10 12:00
comments: true
external-url:
categories: Algorithmic
tags: [algorithmic, dynamic_programming, optimization, complexity]
permalink: /blog/:categories/dynamic-programming
---


> The [memoization]({{ site.baseurl }}/blog/algorithmic/memoization) post ended on a promise : that caching a recursive function opens the door to a whole algorithmic technique called **dynamic programming**. Time to walk through that door. The name is intimidating and, as we will see, deliberately meaningless, but the idea behind it is one you already use when you climb stairs : to know how many ways there are to reach step $n$, you do not re-explore the whole staircase, you look at the two steps you just came from. Dynamic programming is that instinct made rigorous — and this post builds it from an exponential disaster, proves *why* it is allowed to work, fills two tables by hand, and shows exactly where it stops working.

##1. The disaster we are trying to fix
Let us go back to the naive recursive Fibonacci of the [memoization]({{ site.baseurl }}/blog/algorithmic/memoization) post and count precisely how bad it is. Let $T(n)$ be the number of calls made by `fibo_recursive(n)`. The function returns immediately for $n<2$, and otherwise makes one call for $n-1$ and one for $n-2$ :

$$ \begin{equation} T(0) = T(1) = 1, \qquad T(n) = 1 + T(n-1) + T(n-2). \label{eq:calls} \end{equation} $$

That recurrence looks like Fibonacci itself, and it *is* :

<div class="theorem" text="cost of the naive recursion">
	The naive recursive computation of $F(n)$ performs exactly $T(n) = 2F(n+1) - 1$ calls.
	<br/><i>Proof.&nbsp;&nbsp;</i> By strong induction. For $n=0$, $2F(1)-1 = 2\cdot 1 - 1 = 1 = T(0)$, and likewise for $n=1$. Assume the formula holds for $n-1$ and $n-2$. Then by $\eqref{eq:calls}$,
	$$ T(n) = 1 + \big(2F(n)-1\big) + \big(2F(n-1)-1\big) = 2\big(F(n) + F(n-1)\big) - 1 = 2F(n+1) - 1, $$
	using the Fibonacci recurrence itself in the last step.
	<p align="right">$\square$</p>
</div>

Since $F(n) \sim \varphi^{n}/\sqrt{5}$ with $\varphi = \frac{1+\sqrt 5}{2} \approx 1.618$, the number of calls grows **exponentially**. For $n = 50$ that is $2F(51) - 1 \approx 4.1 \times 10^{10}$ calls, hours of computation, to produce a number that a schoolchild could reach in fifty additions.

*In plain words : the recursion is not slow because recursion is slow. It is slow because it keeps forgetting.* Figure 1 shows exactly what it forgets.

<figure><center>
<svg viewBox="0 0 640 360" width="100%" style="max-width:620px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .e{ stroke:#c2ccdc; stroke-width:1.6; fill:none; stroke-linecap:round; }
    .n{ stroke:#4a6da7; stroke-width:1.5; }
    .t{ font-size:12px; fill:#2b2b2b; text-anchor:middle; }
    .cap{ font-size:13.5px; fill:#b5651d; text-anchor:middle; font-style:italic; }
  </style>
  <path class="e" d="M304.9,41.8 L200.1,96.2"/>
  <path class="e" d="M335.4,41.2 L454.6,96.8"/>
  <path class="e" d="M171.9,114.8 L113.1,163.2"/>
  <path class="e" d="M197.6,115.4 L249.4,162.6"/>
  <path class="e" d="M458.4,116.5 L416.6,161.5"/>
  <path class="e" d="M482.7,115.4 L535.3,162.6"/>
  <path class="e" d="M91.3,188.6 L66.7,229.4"/>
  <path class="e" d="M109.9,187.8 L140.1,230.2"/>
  <path class="e" d="M254.6,189.3 L235.4,228.7"/>
  <path class="e" d="M270.1,188.9 L291.9,229.1"/>
  <path class="e" d="M397.4,189.2 L377.6,228.8"/>
  <path class="e" d="M412.9,189.0 L434.1,229.0"/>
  <path class="e" d="M52.3,260.0 L41.7,290.0"/>
  <path class="e" d="M66.2,258.9 L83.8,291.1"/>
  <circle class="n" cx="320" cy="34" r="16" fill="#f0f2f7"></circle><text class="t" x="320" y="39">F5</text>
  <circle class="n" cx="185" cy="104" r="16" fill="#f0f2f7"></circle><text class="t" x="185" y="109">F4</text>
  <circle class="n" cx="470" cy="104" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#e8b48a;#f0f2f7;#f0f2f7;#f0f2f7"/></circle><text class="t" x="470" y="109">F3</text>
  <circle class="n" cx="100" cy="174" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#e8b48a;#f0f2f7;#f0f2f7;#f0f2f7"/></circle><text class="t" x="100" y="179">F3</text>
  <circle class="n" cx="262" cy="174" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#e8b48a;#f0f2f7;#f0f2f7"/></circle><text class="t" x="262" y="179">F2</text>
  <circle class="n" cx="405" cy="174" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#e8b48a;#f0f2f7;#f0f2f7"/></circle><text class="t" x="405" y="179">F2</text>
  <circle class="n" cx="548" cy="174" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#f0f2f7;#e8b48a;#f0f2f7"/></circle><text class="t" x="548" y="179">F1</text>
  <circle class="n" cx="58" cy="244" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#e8b48a;#f0f2f7;#f0f2f7"/></circle><text class="t" x="58" y="249">F2</text>
  <circle class="n" cx="150" cy="244" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#f0f2f7;#e8b48a;#f0f2f7"/></circle><text class="t" x="150" y="249">F1</text>
  <circle class="n" cx="228" cy="244" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#f0f2f7;#e8b48a;#f0f2f7"/></circle><text class="t" x="228" y="249">F1</text>
  <circle class="n" cx="300" cy="244" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#f0f2f7;#f0f2f7;#e8b48a"/></circle><text class="t" x="300" y="249">F0</text>
  <circle class="n" cx="370" cy="244" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#f0f2f7;#e8b48a;#f0f2f7"/></circle><text class="t" x="370" y="249">F1</text>
  <circle class="n" cx="442" cy="244" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#f0f2f7;#f0f2f7;#e8b48a"/></circle><text class="t" x="442" y="249">F0</text>
  <circle class="n" cx="36" cy="306" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#f0f2f7;#e8b48a;#f0f2f7"/></circle><text class="t" x="36" y="311">F1</text>
  <circle class="n" cx="92" cy="306" r="16" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="#f0f2f7;#f0f2f7;#f0f2f7;#e8b48a"/></circle><text class="t" x="92" y="311">F0</text>
  <text class="cap" x="320" y="345" opacity="1">F(3) is recomputed 2 times<animate attributeName="opacity" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="1;0;0;0"/></text>
  <text class="cap" x="320" y="345" opacity="0">F(2) is recomputed 3 times<animate attributeName="opacity" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="0;1;0;0"/></text>
  <text class="cap" x="320" y="345" opacity="0">F(1) is recomputed 5 times<animate attributeName="opacity" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="0;0;1;0"/></text>
  <text class="cap" x="320" y="345" opacity="0">F(0) is recomputed 3 times<animate attributeName="opacity" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.25;0.5;0.75" values="0;0;0;1"/></text>
</svg>
<figcaption><b>Figure 1 -</b> The call tree of $F(5)$ : 15 calls for only 6 distinct values. Watch each value light up everywhere it is recomputed. There are just <b>six</b> different questions in this whole tree, and the recursion asks them fifteen times.</figcaption>
</center></figure>

##1. The key realisation : the tree is really a graph
Here is the whole idea of dynamic programming in one sentence. **The recursion tree of Figure 1 is a lie.** All those nodes labelled `F2` are not different problems that happen to look alike, they are *the same problem*, drawn several times because a tree has no way of showing that two branches meet again.

Glue the identical nodes together and the tree collapses into a **directed acyclic graph** (DAG) of subproblems : one node per distinct question, one edge per dependency.

<figure><center>
<svg viewBox="0 0 640 212" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .ea{ stroke:#4a6da7; stroke-width:1.9; fill:none; }
    .eb{ stroke:#b5651d; stroke-width:1.7; fill:none; opacity:.85; }
    .nd{ stroke:#4a6da7; stroke-width:1.7; }
    .tt{ font-size:12.5px; fill:#2b2b2b; text-anchor:middle; }
    .vv{ font-size:12px; fill:#b5651d; text-anchor:middle; font-weight:bold; }
    .lg{ font-size:12px; fill:#7a7a7a; text-anchor:middle; }
  </style>
  <defs>
    <marker id="ah1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#4a6da7"/></marker>
    <marker id="ah2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#b5651d"/></marker>
  </defs>
  <path class="ea" d="M96.0,100.0 L136.0,100.0" marker-end="url(#ah1)"/>
  <path class="ea" d="M196.0,100.0 L236.0,100.0" marker-end="url(#ah1)"/>
  <path class="ea" d="M296.0,100.0 L336.0,100.0" marker-end="url(#ah1)"/>
  <path class="ea" d="M396.0,100.0 L436.0,100.0" marker-end="url(#ah1)"/>
  <path class="ea" d="M496.0,100.0 L536.0,100.0" marker-end="url(#ah1)"/>
  <path class="eb" d="M84.3,82.1 Q170.0,7.9 254.4,82.1" marker-end="url(#ah2)"/>
  <path class="eb" d="M184.3,117.9 Q270.0,192.1 354.4,117.9" marker-end="url(#ah2)"/>
  <path class="eb" d="M284.3,82.1 Q370.0,7.9 454.4,82.1" marker-end="url(#ah2)"/>
  <path class="eb" d="M384.3,117.9 Q470.0,192.1 554.4,117.9" marker-end="url(#ah2)"/>
  <circle class="nd" cx="70" cy="100" r="23" fill="#f0f2f7"><animate attributeName="fill" dur="7.2s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.167;0.333;0.5;0.667;0.833" values="#e8b48a;#f0f2f7;#f0f2f7;#f0f2f7;#f0f2f7;#f0f2f7"/></circle><text class="tt" x="70" y="99">F0</text><text class="vv" x="70" y="114">0</text>
  <circle class="nd" cx="170" cy="100" r="23" fill="#f0f2f7"><animate attributeName="fill" dur="7.2s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.167;0.333;0.5;0.667;0.833" values="#f0f2f7;#e8b48a;#f0f2f7;#f0f2f7;#f0f2f7;#f0f2f7"/></circle><text class="tt" x="170" y="99">F1</text><text class="vv" x="170" y="114">1</text>
  <circle class="nd" cx="270" cy="100" r="23" fill="#f0f2f7"><animate attributeName="fill" dur="7.2s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.167;0.333;0.5;0.667;0.833" values="#f0f2f7;#f0f2f7;#e8b48a;#f0f2f7;#f0f2f7;#f0f2f7"/></circle><text class="tt" x="270" y="99">F2</text><text class="vv" x="270" y="114">1</text>
  <circle class="nd" cx="370" cy="100" r="23" fill="#f0f2f7"><animate attributeName="fill" dur="7.2s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.167;0.333;0.5;0.667;0.833" values="#f0f2f7;#f0f2f7;#f0f2f7;#e8b48a;#f0f2f7;#f0f2f7"/></circle><text class="tt" x="370" y="99">F3</text><text class="vv" x="370" y="114">2</text>
  <circle class="nd" cx="470" cy="100" r="23" fill="#f0f2f7"><animate attributeName="fill" dur="7.2s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.167;0.333;0.5;0.667;0.833" values="#f0f2f7;#f0f2f7;#f0f2f7;#f0f2f7;#e8b48a;#f0f2f7"/></circle><text class="tt" x="470" y="99">F4</text><text class="vv" x="470" y="114">3</text>
  <circle class="nd" cx="570" cy="100" r="23" fill="#f0f2f7"><animate attributeName="fill" dur="7.2s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.167;0.333;0.5;0.667;0.833" values="#f0f2f7;#f0f2f7;#f0f2f7;#f0f2f7;#f0f2f7;#e8b48a"/></circle><text class="tt" x="570" y="99">F5</text><text class="vv" x="570" y="114">5</text>
  <text class="lg" x="320" y="204">each node holds its value ; the highlight sweeps them in topological order</text>
</svg>
<figcaption><b>Figure 2 -</b> The same computation as a DAG. Every distinct subproblem is one node, and every arrow points from a value towards the value that needs it : blue for the $n-1$ dependency, orange for the $n-2$ one. The orange links alternate above and below the row so that they vault cleanly over the node in between. Fifteen calls became six.</figcaption>
</center></figure>

This reframing is the definition worth memorising :

<div class="definition">
	<b>Dynamic programming</b> is the technique of solving a problem by (i) identifying a family of subproblems that form a <b>directed acyclic graph</b> of dependencies, and (ii) solving each node <b>exactly once</b>, in an order where every node is visited after the nodes it depends on (a <i>topological order</i> of the DAG).
</div>

*In plain words : write down every distinct question you will ever need to answer, notice that they only depend on each other in one direction, then answer them in the right order and write each answer down.* Two ingredients make it possible, and both must be present :

- **Overlapping subproblems.** The same question comes up many times. This is what makes remembering worthwhile — it is why Figure 1 collapses. Without it, storing answers is pure overhead.
- **Optimal substructure.** The optimal answer to a problem can be built from optimal answers to its subproblems. This is what makes remembering *legal* — and, unlike the first, it is a property you must actually prove.

<div class="note">
	This is exactly what separates dynamic programming from <b>divide and conquer</b>. Merge sort also splits a problem into subproblems, but the two halves of an array are disjoint : no subproblem ever reappears, the "DAG" is a genuine tree, and a cache would never get a single hit. Divide and conquer has optimal substructure without overlap ; dynamic programming needs both.
</div>

There are two ways to walk the DAG, and they are the same algorithm seen from two ends :

- **Top-down** (memoization) : keep the recursion, add a dictionary. The call stack discovers the DAG lazily and, by returning from the deepest calls first, ends up visiting nodes in a valid topological order without you ever computing one.
- **Bottom-up** (tabulation) : throw the recursion away, work out the topological order yourself, and fill a table with a loop.

```python
from functools import lru_cache

# top-down : keep the recursion, just add a memory
@lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)

# bottom-up : the same DAG, walked in topological order
def fib_table(n):
    F = [0, 1] + [0] * (n - 1)
    for k in range(2, n + 1):
        F[k] = F[k - 1] + F[k - 2]
    return F[n]
```

Both are $\mathcal{O}(n)$ : each of the $n+1$ nodes is evaluated once, at constant cost. Top-down is easier to write and only ever touches the subproblems it actually needs ; bottom-up avoids the call stack and, as we will see in section 6, makes it obvious which rows of the table can be thrown away.

##1. A real optimisation problem, where greed fails
Fibonacci is a counting problem. Dynamic programming earns its keep on **optimisation** problems, so let us take the smallest honest one.

> **Coin change.** Given a set of coin denominations $S$ (unlimited supply of each) and a target amount $n$, pay exactly $n$ using as **few coins** as possible.

Take $S = \lbrace 1, 3, 4 \rbrace$ and $n = 6$. The obvious strategy is to be **greedy** : always grab the largest coin that still fits. That gives $4 + 1 + 1$, three coins. And it is wrong : $3 + 3$ pays the same amount with two. Greed fails because taking the $4$ is *locally* the best move and *globally* a mistake, and nothing in a greedy algorithm ever revisits that decision. [^1]

So we must consider every first move. Let $C(n)$ be the minimum number of coins for amount $n$. Whatever the optimal solution is, it contains at least one coin ; call it $c$. Remove it, and what remains pays $n - c$. That suggests

$$ \begin{equation} C(0) = 0, \qquad C(n) = 1 + \min_{\substack{c \in S \\ c \le n}} C(n - c). \label{eq:coin} \end{equation} $$

This is the step where beginners are told to "trust the recurrence". Let us instead prove it, because the proof *is* the optimal-substructure property, and it is short.

<div class="theorem" text="optimal substructure of coin change">
	With $C(n)$ the minimum number of coins paying exactly $n$ (and $C(n) = +\infty$ if $n$ cannot be paid), the recurrence $\eqref{eq:coin}$ holds for every $n \ge 1$.
	<br/><br/>
	<i>Proof.&nbsp;&nbsp;</i> We show the two inequalities.
	<br/><b>($\le$)</b> Let $c \in S$ with $c \le n$. Take any optimal payment of $n - c$, which uses $C(n-c)$ coins, and add one coin $c$ to it. The result is a valid payment of $n$ using $C(n-c) + 1$ coins. Since $C(n)$ is the <i>minimum</i> over all valid payments, $C(n) \le 1 + C(n-c)$. This holds for every admissible $c$, hence $C(n) \le 1 + \min_{c} C(n-c)$.
	<br/><b>($\ge$)</b> Take an optimal payment of $n$, using $C(n)$ coins. Because $n \ge 1$ it contains at least one coin ; pick one and call it $c_0$. Removing it leaves a valid payment of $n - c_0$ using $C(n) - 1$ coins. By minimality of $C(n - c_0)$ we get $C(n - c_0) \le C(n) - 1$, that is $C(n) \ge 1 + C(n - c_0) \ge 1 + \min_{c} C(n - c)$.
	<br/>Both inequalities give equality.
	<p align="right">$\square$</p>
</div>

The second half is the classic **cut-and-paste** argument, and it is worth restating in its contrapositive form because that is how you will use it on new problems : *if the remainder of an optimal solution were not itself optimal, we could cut it out, paste in a better sub-solution, and obtain a strictly better solution to the whole problem — contradicting the optimality we assumed.* Every correct dynamic program rests on an argument of this shape.

Now the recurrence is licensed, we can fill the table $C(0), C(1), \dots, C(6)$ in order, each entry needing only entries already written.

<figure><center>
<svg viewBox="0 0 640 248" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .cell{ fill:#f6f7fb; stroke:#4a6da7; stroke-width:1.4; }
    .win{ fill:#fbeede; stroke:#b5651d; stroke-width:2.2; }
    .sweep{ fill:#5aa06a; fill-opacity:.16; stroke:#5aa06a; stroke-width:2.4; }
    .idx{ font-size:12px; fill:#7a7a7a; text-anchor:middle; }
    .val{ font-size:19px; fill:#2b2b2b; text-anchor:middle; }
    .dep{ stroke:#b5651d; stroke-width:1.5; fill:none; opacity:.8; }
    .dlb{ font-size:11.5px; fill:#b5651d; text-anchor:middle;
          paint-order:stroke; stroke:#fff; stroke-width:3.5px; stroke-linejoin:round; }
    .ttl{ font-size:12px; fill:#7a7a7a; text-anchor:middle; }
  </style>
  <defs>
    <marker id="ah3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#b5651d"/></marker>
  </defs>
  <text class="ttl" x="52" y="62">amount n</text>
  <text class="ttl" x="52" y="109">C(n)</text>
  <rect class="cell" x="100" y="74" width="62" height="52"/><text class="idx" x="131" y="62">0</text><text class="val" x="131" y="109">0</text>
  <rect class="cell" x="162" y="74" width="62" height="52"/><text class="idx" x="193" y="62">1</text><text class="val" x="193" y="109">1</text>
  <rect class="cell" x="224" y="74" width="62" height="52"/><text class="idx" x="255" y="62">2</text><text class="val" x="255" y="109">2</text>
  <rect class="cell win" x="286" y="74" width="62" height="52"/><text class="idx" x="317" y="62">3</text><text class="val" x="317" y="109">1</text>
  <rect class="cell" x="348" y="74" width="62" height="52"/><text class="idx" x="379" y="62">4</text><text class="val" x="379" y="109">1</text>
  <rect class="cell" x="410" y="74" width="62" height="52"/><text class="idx" x="441" y="62">5</text><text class="val" x="441" y="109">2</text>
  <rect class="cell win" x="472" y="74" width="62" height="52"/><text class="idx" x="503" y="62">6</text><text class="val" x="503" y="109">2</text>
  <rect class="sweep" x="100" y="74" width="62" height="52"><animate attributeName="x" dur="7s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1250;0.2500;0.3750;0.5000;0.6250;0.7500" values="100;162;224;286;348;410;472"/></rect>
  <path class="dep" d="M481,130 Q461.0,174.0 441,130" marker-end="url(#ah3)"/>
  <path class="dep" d="M500,130 Q408.5,230.0 317,130" marker-end="url(#ah3)"/>
  <path class="dep" d="M519,130 Q387.0,290.0 255,130" marker-end="url(#ah3)"/>
  <text class="dlb" x="415.0" y="146.0">coin 1</text>
  <text class="dlb" x="408.5" y="197.0">coin 3</text>
  <text class="dlb" x="387.0" y="227.0">coin 4</text>
</svg>
<figcaption><b>Figure 3 -</b> The coin-change table for $S=\lbrace 1,3,4 \rbrace$, filled left to right ; the green frame marks the cell being written, and it never comes back. The three arcs below are the only cells $C(6)$ consults, one per coin — they nest instead of crossing. The shaded pair is the winning branch : $C(6) = 1 + C(3) = 2$.</figcaption>
</center></figure>

The winner for $n = 6$ is the $3$-branch : $C(6) = 1 + C(3) = 1 + 1 = 2$, the two threes that greed could not find. The cost is now obvious : $\mathcal{O}(n \cdot \lvert S \rvert)$ time and $\mathcal{O}(n)$ space, against the $\mathcal{O}(\lvert S \rvert^{\,n})$ of the naive recursion.

##1. Recovering the solution, not just its value
The table gives us the *number* $2$, not the coins. This is a general and often-forgotten point : a dynamic program computes an optimal **value**, and getting an optimal **solution** requires remembering, for each state, which choice achieved the minimum. Then you walk those choices backwards from the final state.

```python
def min_coins(coins, amount):
    INF = float("inf")
    C = [0] + [INF] * amount
    choice = [None] * (amount + 1)   # coin achieving the min
    for n in range(1, amount + 1):
        for c in coins:
            if c <= n and C[n - c] + 1 < C[n]:
                C[n], choice[n] = C[n - c] + 1, c
    if C[amount] == INF:
        return None                  # amount cannot be paid
    bag, n = [], amount
    while n:                         # walk the choices back
        bag.append(choice[n])
        n -= choice[n]
    return C[amount], bag

min_coins([1, 3, 4], 6)              # (2, [3, 3])
```

The `choice` array costs one extra number per state and turns an answer into an explanation.

##1. Two dimensions : the knapsack
Not every state is a single integer. Take the other canonical example.

> **0/1 knapsack.** Given $m$ items, item $i$ having weight $w_i$ and value $v_i$, and a bag of capacity $W$, choose a subset of **maximum total value** whose total weight is at most $W$. Each item may be taken at most once.

The "at most once" is what forces a second dimension. If the state were only the remaining capacity, nothing would stop us from taking the same item twice. So the state must record *how far down the item list we are* as well : let $K(i, c)$ be the best value obtainable using only the first $i$ items with capacity $c$. Facing item $i$ we have exactly two options, and by the same cut-and-paste argument the best overall is the better of the two :

$$ \begin{equation} K(i, c) = \max \underbrace{\big\{\, K(i-1,\, c)}_{\text{leave item } i}, \; \underbrace{K(i-1,\, c - w_i) + v_i \,\big\}}_{\text{take it, if } w_i \le c}, \qquad K(0, c) = 0. \label{eq:knap} \end{equation} $$

*In plain words : go through the items one at a time and, for every possible remaining capacity, ask "am I better off skipping this item, or taking it and having less room for the rest ?"* The DAG is now a grid : each cell depends on one cell directly above it and one cell above and to the left.

<figure><center>
<svg viewBox="0 0 640 262" width="100%" style="max-width:620px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .cell2{ fill:#f6f7fb; stroke:#c2ccdc; stroke-width:1.1; }
    .val2{ font-size:14px; fill:#2b2b2b; text-anchor:middle; }
    .hdr{ font-size:12px; fill:#7a7a7a; text-anchor:middle; }
    .rhd{ font-size:12.5px; fill:#4a6da7; text-anchor:end; }
    .sweep2{ fill:#5aa06a; fill-opacity:.13; stroke:#5aa06a; stroke-width:2.2; }
    .mark{ fill:none; stroke:#b5651d; stroke-width:2.2; }
    .ord{ fill:#b5651d; }
    .ordt{ font-size:10px; fill:#fff; text-anchor:middle; font-weight:bold; }
    .cp{ font-size:12px; fill:#7a7a7a; text-anchor:middle; }
  </style>
  <text class="hdr" x="320" y="22">capacity c</text>
  <text class="hdr" x="182" y="48">0</text>
  <text class="hdr" x="234" y="48">1</text>
  <text class="hdr" x="286" y="48">2</text>
  <text class="hdr" x="338" y="48">3</text>
  <text class="hdr" x="390" y="48">4</text>
  <text class="hdr" x="442" y="48">5</text>
  <text class="hdr" x="494" y="48">6</text>
  <text class="hdr" x="546" y="48">7</text>
  <text class="rhd" x="144" y="80">no item</text>
  <rect class="cell2" x="156" y="58" width="52" height="34"/><text class="val2" x="182" y="80">0</text>
  <rect class="cell2" x="208" y="58" width="52" height="34"/><text class="val2" x="234" y="80">0</text>
  <rect class="cell2" x="260" y="58" width="52" height="34"/><text class="val2" x="286" y="80">0</text>
  <rect class="cell2" x="312" y="58" width="52" height="34"/><text class="val2" x="338" y="80">0</text>
  <rect class="cell2" x="364" y="58" width="52" height="34"/><text class="val2" x="390" y="80">0</text>
  <rect class="cell2" x="416" y="58" width="52" height="34"/><text class="val2" x="442" y="80">0</text>
  <rect class="cell2" x="468" y="58" width="52" height="34"/><text class="val2" x="494" y="80">0</text>
  <rect class="cell2" x="520" y="58" width="52" height="34"/><text class="val2" x="546" y="80">0</text>
  <text class="rhd" x="144" y="114">+ (1, 1)</text>
  <rect class="cell2" x="156" y="92" width="52" height="34"/><text class="val2" x="182" y="114">0</text>
  <rect class="cell2" x="208" y="92" width="52" height="34"/><text class="val2" x="234" y="114">1</text>
  <rect class="cell2" x="260" y="92" width="52" height="34"/><text class="val2" x="286" y="114">1</text>
  <rect class="cell2" x="312" y="92" width="52" height="34"/><text class="val2" x="338" y="114">1</text>
  <rect class="cell2" x="364" y="92" width="52" height="34"/><text class="val2" x="390" y="114">1</text>
  <rect class="cell2" x="416" y="92" width="52" height="34"/><text class="val2" x="442" y="114">1</text>
  <rect class="cell2" x="468" y="92" width="52" height="34"/><text class="val2" x="494" y="114">1</text>
  <rect class="cell2" x="520" y="92" width="52" height="34"/><text class="val2" x="546" y="114">1</text>
  <text class="rhd" x="144" y="148">+ (3, 4)</text>
  <rect class="cell2" x="156" y="126" width="52" height="34"/><text class="val2" x="182" y="148">0</text>
  <rect class="cell2" x="208" y="126" width="52" height="34"/><text class="val2" x="234" y="148">1</text>
  <rect class="cell2" x="260" y="126" width="52" height="34"/><text class="val2" x="286" y="148">1</text>
  <rect class="cell2" x="312" y="126" width="52" height="34"/><text class="val2" x="345" y="148">4</text>
  <rect class="cell2" x="364" y="126" width="52" height="34"/><text class="val2" x="390" y="148">5</text>
  <rect class="cell2" x="416" y="126" width="52" height="34"/><text class="val2" x="442" y="148">5</text>
  <rect class="cell2" x="468" y="126" width="52" height="34"/><text class="val2" x="494" y="148">5</text>
  <rect class="cell2" x="520" y="126" width="52" height="34"/><text class="val2" x="546" y="148">5</text>
  <text class="rhd" x="144" y="182">+ (4, 5)</text>
  <rect class="cell2" x="156" y="160" width="52" height="34"/><text class="val2" x="182" y="182">0</text>
  <rect class="cell2" x="208" y="160" width="52" height="34"/><text class="val2" x="234" y="182">1</text>
  <rect class="cell2" x="260" y="160" width="52" height="34"/><text class="val2" x="286" y="182">1</text>
  <rect class="cell2" x="312" y="160" width="52" height="34"/><text class="val2" x="338" y="182">4</text>
  <rect class="cell2" x="364" y="160" width="52" height="34"/><text class="val2" x="390" y="182">5</text>
  <rect class="cell2" x="416" y="160" width="52" height="34"/><text class="val2" x="442" y="182">6</text>
  <rect class="cell2" x="468" y="160" width="52" height="34"/><text class="val2" x="494" y="182">6</text>
  <rect class="cell2" x="520" y="160" width="52" height="34"/><text class="val2" x="553" y="182">9</text>
  <text class="rhd" x="144" y="216">+ (5, 7)</text>
  <rect class="cell2" x="156" y="194" width="52" height="34"/><text class="val2" x="182" y="216">0</text>
  <rect class="cell2" x="208" y="194" width="52" height="34"/><text class="val2" x="234" y="216">1</text>
  <rect class="cell2" x="260" y="194" width="52" height="34"/><text class="val2" x="286" y="216">1</text>
  <rect class="cell2" x="312" y="194" width="52" height="34"/><text class="val2" x="338" y="216">4</text>
  <rect class="cell2" x="364" y="194" width="52" height="34"/><text class="val2" x="390" y="216">5</text>
  <rect class="cell2" x="416" y="194" width="52" height="34"/><text class="val2" x="442" y="216">7</text>
  <rect class="cell2" x="468" y="194" width="52" height="34"/><text class="val2" x="494" y="216">8</text>
  <rect class="cell2" x="520" y="194" width="52" height="34"/><text class="val2" x="553" y="216">9</text>
  <rect class="sweep2" x="156" y="58" width="416" height="34"><animate attributeName="y" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1667;0.3333;0.5000;0.6667" values="58;92;126;160;194"/></rect>
  <rect class="mark" x="314.5" y="128.5" width="47.0" height="29.0" rx="4"/><circle class="ord" cx="325.0" cy="138.0" r="7.5"/><text class="ordt" x="325.0" y="141.5">1</text>
  <rect class="mark" x="522.5" y="162.5" width="47.0" height="29.0" rx="4"/><circle class="ord" cx="533.0" cy="172.0" r="7.5"/><text class="ordt" x="533.0" y="175.5">2</text>
  <rect class="mark" x="522.5" y="196.5" width="47.0" height="29.0" rx="4"/><circle class="ord" cx="533.0" cy="206.0" r="7.5"/><text class="ordt" x="533.0" y="209.5">3</text>
  <text class="cp" x="320" y="250">the sweep goes down one item at a time ; each row reads only the row above it</text>
</svg>
<figcaption><b>Figure 4 -</b> The knapsack table $K(i,c)$ for items $(1,1), (3,4), (4,5), (5,7)$ and capacity $W=7$. Each row consults only the row directly above, so the grid fills in a single downward sweep. The circled cells are the three decisions that build the optimum $9$, numbered in the order you meet them when walking back from the bottom-right corner.</figcaption>
</center></figure>

Read the bottom-right corner : the best value is $\mathbf{9}$. Note *which* items achieve it. Following the choices backwards, $K(4,7) = K(3,7)$ so item $(5,7)$ was **left out** despite being the most valuable single item ; then $K(3,7) \ne K(2,7)$ so item $(4,5)$ is in, dropping us to $K(2,3)$ ; then $K(2,3) \ne K(1,3)$ so item $(3,4)$ is in too. The optimum is $\{(3,4), (4,5)\}$, weight $7$, value $9$ — again a solution no greedy rule based on value, weight, or value-per-weight would produce.

```python
# items = [(weight, value), ...]
def knapsack(items, W):
    K = [[0] * (W + 1) for _ in range(len(items) + 1)]
    for i, (w, v) in enumerate(items, start=1):
        for c in range(W + 1):
            K[i][c] = K[i - 1][c]              # leave it
            if w <= c:                         # or take it
                K[i][c] = max(K[i][c],
                              K[i - 1][c - w] + v)
    return K[len(items)][W]

knapsack([(1, 1), (3, 4), (4, 5), (5, 7)], 7)   # 9
```

##1. Saving space : the rolling table
Look again at $\eqref{eq:knap}$ : row $i$ reads only row $i-1$. Rows $0$ to $i-2$ are dead weight. So we can keep a single array and overwrite it in place, provided we sweep the capacities **downwards** so that `K[c - w]` still holds the *previous* row's value when we read it :

```python
def knapsack_1d(items, W):
    K = [0] * (W + 1)
    for w, v in items:
        # downwards, so K[c - w] still holds row i-1
        for c in range(W, w - 1, -1):
            K[c] = max(K[c], K[c - w] + v)
    return K[W]
```

Memory drops from $\mathcal{O}(mW)$ to $\mathcal{O}(W)$, a change that routinely decides whether a dynamic program fits in RAM. The price is that we can no longer backtrack through the table to recover *which* items were chosen — the same [memoization]({{ site.baseurl }}/blog/algorithmic/memoization) trade-off between speed, memory and information, one level up.

##1. The recipe
Every dynamic program you will ever write is these five steps, in this order.

1. **Name the state.** What is the smallest set of facts that fully describes a subproblem ? ($n$ for coin change ; the pair $(i, c)$ for the knapsack.) Get this wrong and nothing else works.
2. **Write the recurrence.** Consider all the possible *first* (or *last*) decisions and express the answer in terms of smaller states.
3. **Prove optimal substructure.** Cut and paste : assume a sub-solution is not optimal, replace it with a better one, contradict the optimality of the whole.
4. **Order the states.** Find a topological order of the dependency DAG — often just "increasing $n$" or "row by row". Or skip it entirely by memoizing the recursion top-down.
5. **Reconstruct the answer**, if you need the solution and not only its value, by storing the argmin/argmax at each state.

The complexity then reads straight off the table : *(number of states) $\times$ (cost of one transition)*.

##1. Where it breaks, and why that is interesting
Dynamic programming is not universal, and knowing its boundary is what turns it from a trick into a tool.

**Without optimal substructure, it is simply wrong.** The textbook counterexample {% cite Cormen2009 %} is the **longest simple path** between two vertices of a graph. Shortest paths decompose beautifully — a sub-path of a shortest path is a shortest path — which is why Dijkstra and Bellman–Ford work. Longest simple paths do not : the sub-paths of an optimal long path need not be optimal, because "simple" (no repeated vertex) is a *global* constraint that two independently-optimal halves can violate when you glue them together. The problem is NP-hard, and no amount of tabulation will save it.

**The table can be exponentially large.** The knapsack runs in $\mathcal{O}(mW)$, which looks polynomial. It is not : the input writes $W$ in about $\log_2 W$ bits, so the table size $W$ is *exponential in the size of the input*. This is called **pseudo-polynomial**, and it is exactly why the knapsack remains NP-hard even though we just solved it in a dozen lines. The same effect kills exact dynamic programming for the travelling salesman, whose state must record the set of already-visited cities : the Held–Karp algorithm {% cite HeldKarp1962 %} is $\mathcal{O}(2^{m} m^{2})$, a spectacular improvement on the $\mathcal{O}(m!)$ of brute force, and still hopeless past a few dozen cities. Bellman named this state explosion the **curse of dimensionality** {% cite Bellman1957 %}.

**And it is stubbornly sequential.** Each cell waits for its predecessors, which is the exact opposite of the independent-chunks structure that [MapReduce]({{ site.baseurl }}/blog/algorithmic/mapreduce) needs to scale. Dynamic programs parallelise only along the *anti-diagonals* of the table, where cells happen to be mutually independent — never along the direction of the recurrence.

<div class="note">
	<b>Why such an odd name ?</b> Bellman, who founded the field {% cite Bellman1954 %}, chose it for entirely non-mathematical reasons. He was working at RAND under a Secretary of Defense who, in his own words, had "a pathological fear and hatred of the word <i>research</i>" ; "dynamic" sounded impossible to use pejoratively, and "programming" then meant planning, not coding. He admits in his autobiography {% cite Bellman1984 %} that he picked a name nobody could object to, as an umbrella for what he was actually doing. So if the term never made intuitive sense to you : it was never supposed to.
</div>

##1. It is everywhere, once you see the DAG
The pattern generalises far beyond puzzles. Sequence alignment and edit distance fill a two-dimensional grid exactly like the knapsack, and are the backbone of `diff` and of DNA comparison. Viterbi decoding finds the most likely hidden state sequence by tabulating over (time $\times$ state). The deletion–contraction recurrence used to compute [chromatic polynomials]({{ site.baseurl }}/blog/mathematics/introduction-chromatic-polynomials) generates the same graph over and over, and is a textbook candidate for memoization on isomorphism classes.

And one you have already met : [backpropagation]({{ site.baseurl }}/blog/machine-learning/backpropagation). The computational graph of a neural network *is* the DAG, the upstream gradient at each node *is* the memoized subproblem value, and the backward pass *is* the topological sweep. It computes every partial derivative in a single pass for the same reason our table did — because it refuses to answer the same question twice.

##1. Conclusion
Dynamic programming is not a family of tricks to memorise, it is one observation applied with discipline : **a recursion whose calls overlap is secretly a DAG, and a DAG should be evaluated once, in order.** [Memoization]({{ site.baseurl }}/blog/algorithmic/memoization) is that observation applied lazily from the top ; tabulation is the same observation applied deliberately from the bottom. The exponential blow-up of Figure 1 and the polynomial table of Figure 2 describe the *same computation* — the only difference is whether the algorithm bothers to remember.

What makes it an engineering discipline rather than a reflex is the middle step, the one this post insisted on : proving that the pieces of an optimal solution are themselves optimal. When that cut-and-paste argument goes through, you get an algorithm and a proof at the same time. When it does not — longest simple path — no table in the world will rescue you, and that failure is itself worth knowing.


References
----------
{% bibliography --cited %}

[^1]: The number of coins is not the only sensible objective, and the greedy algorithm is not always wrong : for the "canonical" systems used by real currencies (such as $\lbrace 1,2,5,10,20,50 \rbrace$) greed provably matches the optimum. Deciding whether a given coin system is canonical is itself a non-trivial problem — which is a good reason to reach for $\eqref{eq:coin}$ and stop worrying.
