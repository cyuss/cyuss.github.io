---
layout: post
title: Markov Chains and PageRank
date: 2023-05-12 12:00
comments: true
external-url:
categories: Mathematics
tags: [probability, linear_algebra, graphs, algorithms]
permalink: /blog/:categories/markov-chains
---


> In 1913, Andrei Markov took the first 20,000 letters of Pushkin's *Eugene Onegin*, classified each as a vowel or a consonant, and counted how often each followed the other. He was not doing literary criticism. He was making a point in a mathematical argument : that the law of large numbers survives even when observations are **not independent**, provided each one depends only on the one immediately before it {% cite Markov1913 %}. [^1] That single restriction — *the future depends on where you are, not on how you got there* — turned out to describe an enormous amount of the world, and ninety years later it was worth roughly the market capitalisation of Google. This post builds Markov chains from that assumption, works out where they settle down and how fast, and then shows how PageRank is nothing more than one very large chain's stationary distribution.

<div style="text-align:center; margin:2.4em 0 1.2em; color:#b5651d; letter-spacing:.06em;">
  <b>PART I — THE MACHINERY</b><br/>
  <span style="font-size:.85em; color:#7a7a7a; font-style:italic; letter-spacing:0;">one assumption, and the matrix that encodes it</span>
</div>

##1. The memoryless assumption
Imagine a system that moves between a finite set of **states** — sunny, cloudy, rainy ; a page you are reading ; a square on a board. At each step it jumps somewhere according to some probabilities. The Markov assumption says those probabilities depend on **exactly one thing** : where you are right now.

<div class="definition">
	A sequence of random states $X_0, X_1, X_2, \dots$ is a <b>Markov chain</b> if
	$$ \mathbb{P}\big(X_{n+1} = j \;\mid\; X_n = i,\; X_{n-1}, \dots, X_0\big) = \mathbb{P}\big(X_{n+1} = j \;\mid\; X_n = i\big) = p_{ij}. $$
	The entire history before the present moment is irrelevant once you know the present.
</div>

*In plain words : the chain has no memory. It does not know how it arrived, how long it has been wandering, or where it started. It only knows where it is standing, and that is enough to decide where it goes next.*

There is one confusion worth killing immediately, because almost everyone makes it once. **Memoryless does not mean independent.** Tomorrow's weather is strongly dependent on today's — that is the whole point. What the assumption rules out is a dependence on *yesterday's* weather given that you already know today's.

<div class="note">
	You have met this property before in a continuous setting. The <a href="{{ site.baseurl }}/blog/mathematics/probability-distributions">exponential distribution</a> is memoryless in exactly the same sense : having waited twenty minutes tells you nothing about how much longer you will wait. A Markov chain is the discrete-state version of the same idea, and the geometric distribution — how many steps until a chain first leaves a state — is what you get when you ask the waiting-time question here.
</div>

<figure><center>
<svg viewBox="0 0 640 394" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .nd{ fill:#f6f7fb; stroke:#4a6da7; stroke-width:1.8; }
    .nt{ font-size:13px; fill:#2b2b2b; text-anchor:middle; }
    .ed{ stroke:#8fa5c8; stroke-width:1.7; fill:none; }
    .pl{ font-size:11px; fill:#b5651d; text-anchor:middle;
         paint-order:stroke; stroke:#fff; stroke-width:3.5px; stroke-linejoin:round; }
    .cap1{ font-size:11px; fill:#7a7a7a; text-anchor:middle; }
  </style>
  <defs>
    <marker id="ma" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#4a6da7"/></marker>
    <marker id="mo" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#b5651d"/></marker>
  </defs>
  <path class="ed" d="M132.2,275.6 C89.0,293.9 121.6,204.4 139.4,242.6" marker-end="url(#ma)"/>
  <path class="ed" d="M203.1,279.9 Q320.0,306.0 432.0,280.9" marker-end="url(#ma)"/>
  <path class="ed" d="M198.4,252.7 Q269.6,207.4 302.8,135.2" marker-end="url(#ma)"/>
  <path class="ed" d="M436.9,264.1 Q320.0,238.0 208.0,263.1" marker-end="url(#ma)"/>
  <path class="ed" d="M497.1,246.2 C518.4,204.4 551.0,293.9 512.8,276.1" marker-end="url(#ma)"/>
  <path class="ed" d="M456.9,239.3 Q421.6,162.6 354.6,120.0" marker-end="url(#ma)"/>
  <path class="ed" d="M289.6,117.3 Q218.4,162.6 185.2,234.8" marker-end="url(#ma)"/>
  <path class="ed" d="M335.1,130.7 Q370.4,207.4 437.4,250.0" marker-end="url(#ma)"/>
  <path class="ed" d="M304.3,65.6 C272.4,31.3 367.6,31.3 337.8,61.1" marker-end="url(#ma)"/>
  <circle class="nd" cx="320" cy="98" r="36"/>
  <text class="nt" x="320" y="103">sunny</text>
  <circle class="nd" cx="168" cy="272" r="36"/>
  <text class="nt" x="168" y="277">cloudy</text>
  <circle class="nd" cx="472" cy="272" r="36"/>
  <text class="nt" x="472" y="277">rainy</text>
  <text class="pl" x="96.6" y="246.0">0.40</text>
  <text class="pl" x="318.8" y="293.2">0.25</text>
  <text class="pl" x="260.1" y="200.7">0.35</text>
  <text class="pl" x="321.2" y="250.8">0.45</text>
  <text class="pl" x="543.4" y="246.0">0.35</text>
  <text class="pl" x="413.7" y="171.1">0.20</text>
  <text class="pl" x="227.9" y="169.3">0.25</text>
  <text class="pl" x="378.3" y="198.9">0.05</text>
  <text class="pl" x="320.0" y="22.0">0.70</text>
  <text class="cap1" x="320" y="382">every arrow is a probability ; every node's arrows add up to 1</text>
</svg>
<figcaption><b>Figure 1 -</b> A three-state weather chain. Each arrow carries the probability of that move, and the arrows leaving any state sum to one — including the loop that means "no change". Nothing in this picture refers to yesterday : that is the Markov assumption, drawn.</figcaption>
</center></figure>

##1. The transition matrix
Collect all the one-step probabilities $p_{ij}$ into a matrix $P$. Row $i$ holds everything that can happen when you are in state $i$, so **every row sums to one** — a matrix with that property is called *row-stochastic*.

For the weather chain in figure 1, ordering the states as (sunny, cloudy, rainy) :

$$ \begin{equation} P = \begin{pmatrix} 0.70 & 0.25 & 0.05 \\ 0.35 & 0.40 & 0.25 \\ 0.20 & 0.45 & 0.35 \end{pmatrix}. \label{eq:P} \end{equation} $$

Now write the current belief about where you are as a **row vector** $\pi = (\pi_S, \pi_C, \pi_R)$ summing to one. One step forward is a single matrix multiplication :

$$ \begin{equation} \pi_{n+1} = \pi_n P, \qquad\text{and therefore}\qquad \pi_n = \pi_0 P^{\,n}. \label{eq:step} \end{equation} $$

*Check the first one by hand on a single entry. The chance of being cloudy tomorrow is "the chance it is sunny today times the chance sunny leads to cloudy", plus the same for cloudy and for rainy — which is precisely the second component of the product $\pi P$. Matrix multiplication is doing the law of total probability for you, all three states at once.*

The second half of $\eqref{eq:step}$ is more than notation. It says that $n$ steps of a Markov chain is the $n$-th power of a matrix, and it comes with a clean interpretation.

<div class="theorem" text="Chapman–Kolmogorov">
	The entry $(P^{\,n})_{ij}$ is exactly the probability of being in state $j$ after $n$ steps, given that you started in state $i$.
	<br/><br/>
	<i>Proof.&nbsp;&nbsp;</i> By induction. It holds for $n = 1$ by definition. Suppose it holds for $n$. To get from $i$ to $j$ in $n+1$ steps you must pass through some state $k$ at step $n$, and those routes are disjoint, so by the law of total probability
	$$ \mathbb{P}(X_{n+1}=j \mid X_0 = i) = \sum_k \mathbb{P}(X_n = k \mid X_0 = i)\; p_{kj} = \sum_k (P^{\,n})_{ik}\, p_{kj} = (P^{\,n+1})_{ij}, $$
	where the middle step used the Markov property to drop the history before step $n$.
	<p align="right">$\square$</p>
</div>

So the entire long-run behaviour of the chain is a question about **powers of a matrix**, which is a question about its eigenvalues. That is the bridge from probability to linear algebra, and everything in Part II crosses it.

<figure><center>
<svg viewBox="0 0 640 238" width="100%" style="max-width:625px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .mt{ font-size:13px; fill:#4a6da7; text-anchor:middle; }
    .cv{ font-size:10.5px; fill:#2b2b2b; text-anchor:middle; }
    .rw{ font-size:10.5px; fill:#7a7a7a; text-anchor:end; }
    .cap2{ font-size:11px; fill:#7a7a7a; text-anchor:middle; }
    .st{ fill:#b5651d; font-size:12px; }
  </style>
  <text class="mt" x="89.0" y="46">P</text>
  <rect x="32.0" y="66" width="38" height="38" fill="#6969f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="51.0" y="89.0">0.70</text>
  <rect x="70.0" y="66" width="38" height="38" fill="#c9c9f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="89.0" y="89.0">0.25</text>
  <rect x="108.0" y="66" width="38" height="38" fill="#f4f4f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="127.0" y="89.0">0.05</text>
  <text class="rw" x="25.0" y="89.0">S</text>
  <rect x="32.0" y="104" width="38" height="38" fill="#b4b4f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="51.0" y="127.0">0.35</text>
  <rect x="70.0" y="104" width="38" height="38" fill="#a9a9f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="89.0" y="127.0">0.40</text>
  <rect x="108.0" y="104" width="38" height="38" fill="#c9c9f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="127.0" y="127.0">0.25</text>
  <text class="rw" x="25.0" y="127.0">C</text>
  <rect x="32.0" y="142" width="38" height="38" fill="#d4d4f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="51.0" y="165.0">0.20</text>
  <rect x="70.0" y="142" width="38" height="38" fill="#9e9ef2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="89.0" y="165.0">0.45</text>
  <rect x="108.0" y="142" width="38" height="38" fill="#b4b4f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="127.0" y="165.0">0.35</text>
  <text class="rw" x="25.0" y="165.0">R</text>
  <text class="mt" x="235.0" y="46">P²</text>
  <rect x="178.0" y="66" width="38" height="38" fill="#8181f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="197.0" y="89.0">0.59</text>
  <rect x="216.0" y="66" width="38" height="38" fill="#bfbff2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="235.0" y="89.0">0.30</text>
  <rect x="254.0" y="66" width="38" height="38" fill="#e6e6f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="273.0" y="89.0">0.12</text>
  <rect x="178.0" y="104" width="38" height="38" fill="#a1a1f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="197.0" y="127.0">0.43</text>
  <rect x="216.0" y="104" width="38" height="38" fill="#b1b1f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="235.0" y="127.0">0.36</text>
  <rect x="254.0" y="104" width="38" height="38" fill="#d3d3f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="273.0" y="127.0">0.21</text>
  <rect x="178.0" y="142" width="38" height="38" fill="#b0b0f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="197.0" y="165.0">0.37</text>
  <rect x="216.0" y="142" width="38" height="38" fill="#ababf2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="235.0" y="165.0">0.39</text>
  <rect x="254.0" y="142" width="38" height="38" fill="#cacaf2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="273.0" y="165.0">0.24</text>
  <text class="mt" x="381.0" y="46">P⁵</text>
  <rect x="324.0" y="66" width="38" height="38" fill="#9292f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="343.0" y="89.0">0.51</text>
  <rect x="362.0" y="66" width="38" height="38" fill="#b8b8f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="381.0" y="89.0">0.33</text>
  <rect x="400.0" y="66" width="38" height="38" fill="#dcdcf2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="419.0" y="89.0">0.16</text>
  <rect x="324.0" y="104" width="38" height="38" fill="#9595f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="343.0" y="127.0">0.49</text>
  <rect x="362.0" y="104" width="38" height="38" fill="#b7b7f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="381.0" y="127.0">0.34</text>
  <rect x="400.0" y="104" width="38" height="38" fill="#dadaf2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="419.0" y="127.0">0.17</text>
  <rect x="324.0" y="142" width="38" height="38" fill="#9696f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="343.0" y="165.0">0.49</text>
  <rect x="362.0" y="142" width="38" height="38" fill="#b6b6f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="381.0" y="165.0">0.34</text>
  <rect x="400.0" y="142" width="38" height="38" fill="#d9d9f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="419.0" y="165.0">0.17</text>
  <text class="mt" x="527.0" y="46">P²⁰</text>
  <rect x="470.0" y="66" width="38" height="38" fill="#9393f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="489.0" y="89.0">0.50</text>
  <rect x="508.0" y="66" width="38" height="38" fill="#b7b7f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="527.0" y="89.0">0.33</text>
  <rect x="546.0" y="66" width="38" height="38" fill="#dbdbf2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="565.0" y="89.0">0.17</text>
  <rect x="470.0" y="104" width="38" height="38" fill="#9393f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="489.0" y="127.0">0.50</text>
  <rect x="508.0" y="104" width="38" height="38" fill="#b7b7f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="527.0" y="127.0">0.33</text>
  <rect x="546.0" y="104" width="38" height="38" fill="#dbdbf2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="565.0" y="127.0">0.17</text>
  <rect x="470.0" y="142" width="38" height="38" fill="#9393f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="489.0" y="165.0">0.50</text>
  <rect x="508.0" y="142" width="38" height="38" fill="#b7b7f2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="527.0" y="165.0">0.33</text>
  <rect x="546.0" y="142" width="38" height="38" fill="#dbdbf2" stroke="#dfe4ee" stroke-width="1"/>
  <text class="cv" x="565.0" y="165.0">0.17</text>
  <text class="cap2" x="320" y="206">the three rows start different — where you are today matters — and end identical</text>
  <text class="cap2 st" x="320" y="226">that shared row is the stationary distribution π ≈ (0.495, 0.323, 0.182)</text>
</svg>
<figcaption><b>Figure 2 -</b> Powers of the transition matrix. Entry $(i,j)$ of $P^n$ is the chance of being in state $j$ after $n$ steps having started in $i$. After twenty steps every row is the same, which is the chain saying it has completely forgotten where it began.</figcaption>
</center></figure>

<div style="text-align:center; margin:2.4em 0 1.2em; color:#b5651d; letter-spacing:.06em;">
  <b>PART II — THE LONG RUN</b><br/>
  <span style="font-size:.85em; color:#7a7a7a; font-style:italic; letter-spacing:0;">where it settles, whether that is unique, and how fast it gets there</span>
</div>

##1. The stationary distribution
Figure 2 shows something remarkable : as you raise $P$ to higher powers, the rows become identical. That means the chain forgets its starting point entirely. Whatever the weather today, the probability that it is raining a month from now is the same number.

<div class="definition">
	A distribution $\pi$ is <b>stationary</b> if running one more step changes nothing :
	$$ \pi = \pi P, \qquad \sum_i \pi_i = 1, \qquad \pi_i \ge 0. $$
</div>

Read that equation again with linear-algebra eyes. It says $\pi$ is a **left eigenvector of $P$ with eigenvalue 1** — and $1$ is always an eigenvalue of a row-stochastic matrix, because the all-ones column vector satisfies $P\mathbf{1} = \mathbf{1}$ (each row sums to one). So a stationary distribution always exists for a finite chain.

*In plain words : the stationary distribution is the mixture that reproduces itself. Put 60 % of your probability on sunny and the rest split appropriately, run one day forward, and you get the identical mixture back. It is the equilibrium the system relaxes into.*

For the weather chain, solving $\pi = \pi P$ together with $\sum \pi_i = 1$ gives

$$ \pi \;\approx\; (0.4949,\; 0.3232,\; 0.1818), $$

so in the long run it is sunny about half the time, cloudy a third, rainy a sixth — regardless of what it is doing today. There is a second, very useful reading of the same numbers : $\pi_i$ is the **fraction of time** the chain spends in state $i$ over a long run, and $1/\pi_i$ is the average number of steps between successive visits to it.

##1. When does it actually converge ?
Here is where most treatments wave their hands, and where the interesting failures live. Existence of $\pi$ is free. **Uniqueness** and **convergence** are not, and each needs its own condition.

<div class="definition">
	A chain is <b>irreducible</b> if you can get from any state to any other state in some number of steps — the graph is one connected piece.
	<br/><br/>
	A chain is <b>aperiodic</b> if it is not locked into a cycle : formally, the greatest common divisor of the possible return times to a state is 1.
</div>

<div class="theorem" text="the fundamental theorem for finite chains">
	A finite Markov chain that is <b>irreducible</b> and <b>aperiodic</b> has a <b>unique</b> stationary distribution $\pi$, and from <i>any</i> starting distribution
	$$ \pi_0 P^{\,n} \;\longrightarrow\; \pi \qquad\text{as } n \to \infty. $$
	Drop either condition and the conclusion fails, in two quite different ways.
</div>

<figure><center>
<svg viewBox="0 0 640 300" width="100%" style="max-width:625px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .nd{ fill:#f6f7fb; stroke:#4a6da7; stroke-width:1.6; }
    .nt{ font-size:12.5px; fill:#2b2b2b; text-anchor:middle; }
    .ed{ stroke:#8fa5c8; stroke-width:1.7; fill:none; }
    .or{ stroke:#dda87a; }
    .dv{ stroke:#e2e2e2; stroke-width:1.2; }
    .ht{ font-size:12.5px; text-anchor:middle; font-weight:bold; }
    .bad{ fill:#b5651d; }
    .hs{ font-size:10.5px; fill:#7a7a7a; text-anchor:middle; font-style:italic; }
    .cc{ font-size:10.5px; fill:#5a5a5a; text-anchor:middle; }
  </style>
  <defs>
    <marker id="ma" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#4a6da7"/></marker>
    <marker id="mo" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#b5651d"/></marker>
  </defs>
  <line class="dv" x1="320" y1="40" x2="320" y2="252"/>
  <path class="ed" d="M108.8,130.5 Q141.0,148.0 168.8,132.8" marker-end="url(#ma)"/>
  <path class="ed" d="M173.2,105.5 Q141.0,88.0 113.2,103.2" marker-end="url(#ma)"/>
  <path class="ed" d="M108.8,228.5 Q141.0,246.0 168.8,230.8" marker-end="url(#ma)"/>
  <path class="ed" d="M173.2,203.5 Q141.0,186.0 113.2,201.2" marker-end="url(#ma)"/>
  <circle class="nd" cx="86" cy="118" r="26"/><text class="nt" x="86" y="123">A</text>
  <circle class="nd" cx="196" cy="118" r="26"/><text class="nt" x="196" y="123">B</text>
  <circle class="nd" cx="86" cy="216" r="26"/><text class="nt" x="86" y="221">C</text>
  <circle class="nd" cx="196" cy="216" r="26"/><text class="nt" x="196" y="221">D</text>
  <text class="ht bad" x="141" y="66">not irreducible</text>
  <text class="hs" x="141" y="82">two islands, no way across</text>
  <text class="cc" x="141" y="270">π exists but is <tspan font-style="italic">not unique</tspan> —</text>
  <text class="cc" x="141" y="286">the answer depends on where you started</text>
  <path class="ed or" d="M488.7,142.5 Q503.2,183.2 532.4,208.0" marker-end="url(#mo)"/>
  <path class="ed or" d="M530.7,222.0 Q480.0,210.0 434.2,220.9" marker-end="url(#mo)"/>
  <path class="ed or" d="M423.8,211.2 Q456.8,183.2 469.6,147.2" marker-end="url(#mo)"/>
  <circle class="nd" cx="480" cy="118" r="26"/><text class="nt" x="480" y="123">X</text>
  <circle class="nd" cx="556" cy="228" r="26"/><text class="nt" x="556" y="233">Y</text>
  <circle class="nd" cx="404" cy="228" r="26"/><text class="nt" x="404" y="233">Z</text>
  <text class="ht bad" x="480" y="62">not aperiodic</text>
  <text class="hs" x="480" y="80">every step must advance the cycle</text>
  <text class="cc" x="480" y="270">π <tspan font-style="italic">is</tspan> unique, but Pⁿ never converges —</text>
  <text class="cc" x="480" y="286">the distribution cycles with period 3 for ever</text>
</svg>
<figcaption><b>Figure 3 -</b> The two ways the fundamental theorem fails, and they fail differently. On the left the chain cannot cross between the halves, so each half has its own equilibrium and there is no single answer. On the right there is a unique equilibrium, but the chain is locked into a cycle and marches round it for ever without ever settling into it. One self-loop anywhere on the right would fix it.</figcaption>
</center></figure>

**Lose irreducibility and uniqueness goes.** If the graph splits into two pieces that cannot reach each other, each piece has its own equilibrium, and any blend of the two is also stationary. There is no single answer to "where does it end up" because the answer depends on which half you started in. This is not an exotic corner case — it is exactly what the web looks like, and section 6 is about the trick that repairs it.

**Lose aperiodicity and convergence goes, even though $\pi$ is still unique.** Take a chain that must move $A \to B \to C \to A$ every step. Start at $A$ and after $n$ steps you are at $A$, $B$ or $C$ depending only on $n \bmod 3$ — the distribution oscillates forever and never settles. The stationary distribution $(1/3, 1/3, 1/3)$ is perfectly well defined and describes the long-run *time average*, but $P^n$ does not converge to anything. Adding a single self-loop anywhere breaks the cycle and fixes it.

##1. How fast ? The second eigenvalue
Convergence is guaranteed, but a guarantee with no rate is useless in practice. The rate is controlled by one number.

For an irreducible aperiodic chain the eigenvalues satisfy $\lambda_1 = 1 > \lvert\lambda_2\rvert \ge \lvert\lambda_3\rvert \ge \cdots$, and the distance to equilibrium decays like the second-largest one :

$$ \begin{equation} \big\lVert \pi_0 P^{\,n} - \pi \big\rVert \;\le\; C\,\lvert\lambda_2\rvert^{\,n}. \label{eq:gap} \end{equation} $$

The quantity $1 - \lvert\lambda_2\rvert$ is the **spectral gap**, and the number of steps needed to get close to equilibrium — the **mixing time** — scales roughly as $1/(1 - \lvert\lambda_2\rvert)$ {% cite Levin2017 %}.

*In plain words : $\lambda_2$ measures how bottlenecked the chain is. If the state space has two well-connected regions joined by one narrow bridge, the chain will equilibrate quickly inside each region and then take a very long time to balance across the bridge. That slow crossing is $\lambda_2$, and it is close to 1. A chain that mixes everything freely has a small $\lvert\lambda_2\rvert$ and forgets its start almost immediately.*

<figure><center>
<svg viewBox="0 0 640 274" width="100%" style="max-width:620px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .gr{ stroke:#ececec; stroke-width:1; }
    .ax{ stroke:#bdbdbd; stroke-width:1.3; }
    .c-fast{ fill:none; stroke:#5aa06a; stroke-width:2.3; }
    .c-slow{ fill:none; stroke:#b5651d; stroke-width:2.3; }
    .lb{ font-size:11.5px; stroke:none;
         paint-order:stroke; stroke:#fff; stroke-width:3.5px; stroke-linejoin:round; }
    text.c-fast{ fill:#5aa06a; } text.c-slow{ fill:#b5651d; }
    .tk{ font-size:10.5px; fill:#7a7a7a; text-anchor:end; }
    .mid{ text-anchor:middle; }
    .axl{ font-size:11.5px; fill:#5a5a5a; text-anchor:middle; }
  </style>
  <line class="gr" x1="92" y1="48.0" x2="596" y2="48.0"/>
  <text class="tk" x="84" y="52.0">1</text>
  <line class="gr" x1="92" y1="77.7" x2="596" y2="77.7"/>
  <text class="tk" x="84" y="81.7">10⁻1</text>
  <line class="gr" x1="92" y1="107.3" x2="596" y2="107.3"/>
  <text class="tk" x="84" y="111.3">10⁻2</text>
  <line class="gr" x1="92" y1="137.0" x2="596" y2="137.0"/>
  <text class="tk" x="84" y="141.0">10⁻3</text>
  <line class="gr" x1="92" y1="166.7" x2="596" y2="166.7"/>
  <text class="tk" x="84" y="170.7">10⁻4</text>
  <line class="gr" x1="92" y1="196.3" x2="596" y2="196.3"/>
  <text class="tk" x="84" y="200.3">10⁻5</text>
  <line class="gr" x1="92" y1="226.0" x2="596" y2="226.0"/>
  <text class="tk" x="84" y="230.0">10⁻6</text>
  <text class="tk mid" x="92.0" y="244">0</text>
  <text class="tk mid" x="218.0" y="244">30</text>
  <text class="tk mid" x="344.0" y="244">60</text>
  <text class="tk mid" x="470.0" y="244">90</text>
  <text class="tk mid" x="596.0" y="244">120</text>
  <path class="c-fast" d="M92.0,48.0 L96.2,63.5 L100.4,79.0 L104.6,94.5 L108.8,110.0 L113.0,125.6 L117.2,141.1 L121.4,156.6 L125.6,172.1 L129.8,187.6 L134.0,203.1 L138.2,218.6"/>
  <text class="lb c-fast" x="110.2" y="55.5">well mixed :  |λ₂| = 0.30</text>
  <path class="c-slow" d="M92.0,48.0 L96.2,48.7 L100.4,49.3 L104.6,50.0 L108.8,50.6 L113.0,51.3 L117.2,52.0 L121.4,52.6 L125.6,53.3 L129.8,53.9 L134.0,54.6 L138.2,55.3 L142.4,55.9 L146.6,56.6 L150.8,57.3 L155.0,57.9 L159.2,58.6 L163.4,59.2 L167.6,59.9 L171.8,60.6 L176.0,61.2 L180.2,61.9 L184.4,62.5 L188.6,63.2 L192.8,63.9 L197.0,64.5 L201.2,65.2 L205.4,65.8 L209.6,66.5 L213.8,67.2 L218.0,67.8 L222.2,68.5 L226.4,69.1 L230.6,69.8 L234.8,70.5 L239.0,71.1 L243.2,71.8 L247.4,72.5 L251.6,73.1 L255.8,73.8 L260.0,74.4 L264.2,75.1 L268.4,75.8 L272.6,76.4 L276.8,77.1 L281.0,77.7 L285.2,78.4 L289.4,79.1 L293.6,79.7 L297.8,80.4 L302.0,81.0 L306.2,81.7 L310.4,82.4 L314.6,83.0 L318.8,83.7 L323.0,84.3 L327.2,85.0 L331.4,85.7 L335.6,86.3 L339.8,87.0 L344.0,87.7 L348.2,88.3 L352.4,89.0 L356.6,89.6 L360.8,90.3 L365.0,91.0 L369.2,91.6 L373.4,92.3 L377.6,92.9 L381.8,93.6 L386.0,94.3 L390.2,94.9 L394.4,95.6 L398.6,96.2 L402.8,96.9 L407.0,97.6 L411.2,98.2 L415.4,98.9 L419.6,99.5 L423.8,100.2 L428.0,100.9 L432.2,101.5 L436.4,102.2 L440.6,102.9 L444.8,103.5 L449.0,104.2 L453.2,104.8 L457.4,105.5 L461.6,106.2 L465.8,106.8 L470.0,107.5 L474.2,108.1 L478.4,108.8 L482.6,109.5 L486.8,110.1 L491.0,110.8 L495.2,111.4 L499.4,112.1 L503.6,112.8 L507.8,113.4 L512.0,114.1 L516.2,114.7 L520.4,115.4 L524.6,116.1 L528.8,116.7 L533.0,117.4 L537.2,118.1 L541.4,118.7 L545.6,119.4 L549.8,120.0 L554.0,120.7 L558.2,121.4 L562.4,122.0 L566.6,122.7 L570.8,123.3 L575.0,124.0 L579.2,124.7 L583.4,125.3 L587.6,126.0 L591.8,126.6 L596.0,127.3"/>
  <text class="lb c-slow" x="416.8" y="88.9">bottlenecked :  |λ₂| = 0.95</text>
  <line class="ax" x1="92" y1="226" x2="596" y2="226"/>
  <line class="ax" x1="92" y1="48" x2="92" y2="226"/>
  <text class="axl" x="344" y="262">number of steps</text>
  <text class="axl" x="16" y="137" transform="rotate(-90 16,137)">distance from equilibrium</text>
</svg>
<figcaption><b>Figure 4 -</b> Two chains with the same equilibrium, converging at wildly different speeds. On this logarithmic scale both are straight lines, and the slope is $\lvert\lambda_2\rvert$. The well-mixed chain is within one part in a million after twelve steps ; the bottlenecked one needs about 270. Same answer, three orders of magnitude apart in cost.</figcaption>
</center></figure>

This is not an abstract concern. It is the single most important practical question about any chain you build : in section 8, it decides whether your Bayesian sampler gives you an answer in an hour or in a geological age.

<div style="text-align:center; margin:2.4em 0 1.2em; color:#b5651d; letter-spacing:.06em;">
  <b>PART III — WHAT IT IS FOR</b><br/>
  <span style="font-size:.85em; color:#7a7a7a; font-style:italic; letter-spacing:0;">ranking the web, hidden states, and turning the whole thing around</span>
</div>

##1. PageRank : a chain the size of the web
Here is the idea that made Part II worth a fortune. Model the web as a graph : pages are states, links are transitions. Now imagine a **random surfer** who starts anywhere and repeatedly clicks a link chosen uniformly at random from the current page {% cite Brin1998 %}{% cite Page1999 %}.

Where does this surfer spend their time ? That is exactly the stationary distribution question from section 3 — and its answer is a ranking. A page is important if the surfer is often on it, which happens when many pages link to it, and especially when *important* pages link to it. The definition is recursive, which sounds circular and is in fact just an eigenvector equation.

There are two problems, and both are failures of the conditions in section 4.

- **Dangling pages.** A page with no outgoing links is a dead end : the surfer arrives and the probability leaks out of the system. The row of $P$ is all zeros and it is not a stochastic matrix any more.
- **The web is not irreducible.** It is full of pieces that link inward but never back out, so by section 4 there is no unique answer.

The fix is one line, and it repairs both at once. With probability $d \approx 0.85$ the surfer follows a link ; with probability $1-d$ they get bored and **teleport** to a page chosen uniformly at random :

$$ \begin{equation} G \;=\; d\,P \;+\; \frac{1-d}{N}\,J, \qquad J = \text{the all-ones matrix.} \label{eq:google} \end{equation} $$

<div class="theorem" text="why damping works">
	Every entry of $G$ is at least $(1-d)/N > 0$. A matrix with all entries strictly positive describes a chain that can reach any state from any state in one step, so $G$ is <b>irreducible</b>, and it has self-loops, so it is <b>aperiodic</b>. By the theorem of section 4 its stationary distribution exists, is unique, and is reached from any starting point. Moreover $\lvert\lambda_2(G)\rvert \le d$, so convergence is geometric at rate $0.85$ — about 50 to 100 iterations for web-scale accuracy, whatever the size of the web.
</div>

*In plain words : the teleport is not a hack bolted on to fix bad data. It is the thing that makes the question well posed at all, and the damping factor is simultaneously the knob that guarantees the answer is unique and the knob that controls how fast you can compute it.* This is a genuinely satisfying case of Perron–Frobenius theory {% cite Frobenius1912 %} paying rent.

Computing it needs no eigenvalue solver. Just apply $\eqref{eq:step}$ repeatedly — **power iteration** — and $\eqref{eq:gap}$ guarantees you converge.

```python
def pagerank(links, n, d=0.85, iters=60):
    """links[i] = list of pages that page i points to."""
    out = [len(links[i]) for i in range(n)]
    rank = [1.0 / n] * n
    for _ in range(iters):
        new = [(1 - d) / n] * n
        leak = 0.0
        for i in range(n):
            if out[i] == 0:
                leak += d * rank[i] / n      # dangling page
            else:
                share = d * rank[i] / out[i]
                for j in links[i]:
                    new[j] += share
        rank = [r + leak for r in new]
    return rank
```

<figure><center>
<svg viewBox="0 0 640 364" width="100%" style="max-width:620px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .nd{ stroke-width:1.8; }
    .small{ fill:#eef1f8; stroke:#8fa5c8; }
    .big{ fill:#fbeede; stroke:#b5651d; }
    circle.big{ stroke-width:2.2; }
    .nt{ font-size:13px; fill:#2b2b2b; text-anchor:middle; }
    .rk{ font-size:10.5px; text-anchor:middle; }
    text.small{ fill:#7a7a7a; } text.big{ fill:#b5651d; font-size:12px; }
    .ed{ stroke:#a9b8d0; stroke-width:1.7; fill:none; }
    .hot{ stroke:#dda87a; stroke-width:2.2; }
    .an{ font-size:11px; fill:#7a7a7a; text-anchor:middle; font-style:italic; stroke:none; }
    text.hot{ fill:#b5651d; stroke:none; }
    .cap5{ font-size:11px; fill:#7a7a7a; text-anchor:middle; }
  </style>
  <defs>
    <marker id="ma" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#4a6da7"/></marker>
    <marker id="mo" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#b5651d"/></marker>
  </defs>
  <path class="ed" d="M90.5,69.7 Q175.0,122.0 228.6,155.2" marker-end="url(#ma)"/>
  <path class="ed" d="M92.4,145.0 Q175.0,162.0 222.1,171.7" marker-end="url(#ma)"/>
  <path class="ed" d="M92.4,219.0 Q175.0,202.0 222.1,192.3" marker-end="url(#ma)"/>
  <path class="ed" d="M90.5,294.3 Q175.0,242.0 228.6,208.8" marker-end="url(#ma)"/>
  <path class="ed hot" d="M314.5,199.5 Q379.0,226.0 440.5,200.7" marker-end="url(#mo)"/>
  <path class="ed hot" d="M445.1,165.2 Q379.0,138.0 319.2,162.6" marker-end="url(#mo)"/>
  <circle class="nd small" cx="78" cy="62" r="14.7"/>
  <text class="nt" x="78" y="67">A</text>
  <text class="rk small" x="78" y="91.7">0.025</text>
  <circle class="nd small" cx="78" cy="142" r="14.7"/>
  <text class="nt" x="78" y="147">B</text>
  <text class="rk small" x="78" y="171.7">0.025</text>
  <circle class="nd small" cx="78" cy="222" r="14.7"/>
  <text class="nt" x="78" y="227">C</text>
  <text class="rk small" x="78" y="251.7">0.025</text>
  <circle class="nd small" cx="78" cy="302" r="14.7"/>
  <text class="nt" x="78" y="307">D</text>
  <text class="rk small" x="78" y="331.7">0.025</text>
  <circle class="nd big" cx="272" cy="182" r="46.0"/>
  <text class="nt" x="272" y="187">H</text>
  <text class="rk big" x="272" y="243.0">0.473</text>
  <circle class="nd big" cx="486" cy="182" r="44.2"/>
  <text class="nt" x="486" y="187">P</text>
  <text class="rk big" x="486" y="241.2">0.427</text>
  <text class="an" x="78" y="24">four unremarkable pages …</text>
  <text class="an hot" x="272" y="90">… all point here</text>
  <text class="an hot" x="486" y="90">one link in, nearly the same rank</text>
  <text class="cap5" x="320" y="352">a link is a vote, and votes are weighted by the voter's own rank</text>
</svg>
<figcaption><b>Figure 5 -</b> PageRank on six pages, with each circle sized by its score. A, B, C and D have nothing pointing at them, so they sit at the floor value $(1-d)/N$. They all point at H, which collects their weight. H then points at a single page, P — and P, with exactly one incoming link, scores almost as highly as H with its five. The recursion is the point : it is not how many links you get, it is whose.</figcaption>
</center></figure>

Figure 5 makes the recursive definition concrete. Four unremarkable pages point at the hub, so the hub accumulates their weight. The hub then points at a single page — and that page, with **one** incoming link, ends up ranked almost as highly as the hub with its five. That is the whole insight : links are votes, but votes are weighted by the voter's own standing.

<div class="note">
	At web scale this is a matrix with billions of rows, and each iteration is a sparse matrix–vector product — a sum over all links, grouped by destination. That shape is precisely a <a href="{{ site.baseurl }}/blog/algorithmic/mapreduce">MapReduce</a> job : map each link to a contribution keyed by the target page, reduce by summing. PageRank was for years the canonical example in every MapReduce tutorial, for exactly this reason.
</div>

##1. When you cannot see the state
Often the chain is real but invisible. You do not observe the states, only noisy consequences of them : you hear sounds, not phonemes ; you read words, not grammatical categories ; you sequence DNA, not the genes' functional regions. A **hidden Markov model** adds a second layer — a Markov chain over hidden states, plus an emission distribution giving what you observe from each {% cite Rabiner1989 %}.

Three questions follow, and each has a clean algorithm. How likely is this observation sequence ? (the forward algorithm.) What is the most probable hidden path that produced it ? (the **Viterbi** algorithm {% cite Viterbi1967 %}.) And how do I learn the parameters from data alone ? (Baum–Welch.)

<div class="note">
	Viterbi is <a href="{{ site.baseurl }}/blog/algorithmic/dynamic-programming">dynamic programming</a>, exactly as that post describes it. The state is "the best path ending in hidden state $j$ at time $t$", the recurrence maximises over the previous state, and the optimal-substructure argument is the standard cut-and-paste : if the best path to $j$ at time $t$ did not use the best path to some $k$ at time $t-1$, you could swap it in and do better. Naively there are exponentially many paths ; the table has $T \times S$ cells.
</div>

##1. Turning the whole thing around : MCMC
Everything so far asked : *given a chain, what does it converge to ?* One of the best ideas in computational statistics is to run that question backwards. **Given a distribution you want to sample from, design a chain that converges to it.**

This matters because in Bayesian [inference]({{ site.baseurl }}/blog/mathematics/statistical-inference) you routinely end up with a posterior you can evaluate up to an unknown constant but cannot integrate, sample, or even normalise. Metropolis and colleagues {% cite Metropolis1953 %}, later generalised by Hastings {% cite Hastings1970 %}, found a chain whose stationary distribution is any target $p$ you like :

1. From the current point $x$, propose a nearby $x'$.
2. Accept it with probability $\min\!\big(1,\; p(x')/p(x)\big)$ ; otherwise stay at $x$.

The unknown normalising constant cancels in the ratio, which is the trick. The chain provably has $p$ as its stationary distribution, so after enough steps its visits are draws from $p$ — and "enough steps" is the mixing time of section 5. A badly designed proposal makes $\lvert\lambda_2\rvert$ close to 1, the chain crawls, and you get thousands of samples that are all essentially the same point. Every practical difficulty with MCMC is a spectral gap problem in disguise.

##1. Where the assumption breaks
Real sequences have memory. Language certainly does : the word after *"the capital of France is"* depends on all six words, not just on *"is"*.

The standard repair is to **enlarge the state** until the assumption becomes true. Make the state the last $k$ words rather than the last one, and you have an $n$-gram model, which is genuinely Markov again. The cost is brutal : with a vocabulary of $V$ words the state space is $V^{k}$, so it explodes exponentially and you never have enough data to estimate the transitions. This is why $n$-gram language models stalled at $k = 5$ or so for decades.

<div class="note">
	This is precisely the limitation that <a href="{{ site.baseurl }}/blog/machine-learning/transformers">Transformers</a> escape. <a href="{{ site.baseurl }}/blog/machine-learning/attention-mechanism">Attention</a> lets every position look directly at every earlier position, so the model conditions on the whole history <i>without</i> having to encode that history into a state whose size explodes. Seen this way, the move from $n$-grams to attention is the move from "make the state big enough to be Markov" to "stop being Markov".
</div>

Two other honest limitations. Real chains are often **non-stationary** — the transition probabilities themselves drift over time, and a model fitted last year quietly stops applying. And the discrete-state picture forces you to bin continuous quantities, which is a modelling choice that can matter more than the chain itself.

##1. Conclusion
A Markov chain is one assumption — *the future depends only on the present* — and everything else is consequence.

Encode the assumption as a matrix and $n$ steps become the $n$-th power, which turns a probabilistic question into a question about eigenvalues. The eigenvalue $1$ gives the equilibrium the chain relaxes into ; irreducibility makes that equilibrium unique ; aperiodicity makes the chain actually reach it ; and the second eigenvalue says how long you will be waiting. Those four facts are the whole theory, and they are enough to explain why adding a 15 % chance of teleporting to a random page is what makes ranking the entire web both well defined and computable in fifty iterations.

The assumption is also a design choice, not a law of nature. When it is false, you can enlarge the state until it becomes true and pay in exponential blow-up — or you can abandon it, which is the road that led to attention. Knowing which of those you are doing, and why, is most of the value in understanding the chain in the first place.


References
----------
{% bibliography --cited %}

[^1]: Markov's choice of *Eugene Onegin* was polemical. He was in a long argument with Pavel Nekrasov, who had claimed that the law of large numbers required independence and therefore that its appearance in social statistics was evidence of free will. Markov's vowel-and-consonant counts were a deliberate counterexample : here is a sequence that is plainly *not* independent, and the law of large numbers holds anyway. The theory of stochastic processes began as a rebuttal in a philosophical dispute.
