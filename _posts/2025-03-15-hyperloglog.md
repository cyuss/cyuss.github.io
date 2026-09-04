---
layout: post
title: HyperLogLog, Counting Millions with Kilobytes
date: 2025-03-15 15:00
comments: true
external-url:
categories: Algorithmic
tags: [algorithmic, probability, data_structures]
permalink: /blog/:categories/hyperloglog
---


> How many **distinct** visitors did your website get today ? The obvious way is to keep a set of every visitor ID you have seen and ask for its size, but at the scale of billions of events that set would swallow gigabytes of memory. **HyperLogLog** {% cite Flajolet2007 %} performs the same count, distinct elements in a massive stream, using a *few kilobytes*, by giving up exactness for a tiny, controllable error. Its secret is a delightful piece of probability : rare coincidences betray how many things you have seen. If this trade, sacrifice exactness to save enormous space, sounds familiar, it is the same bargain struck by [Bloom filters]({{ site.baseurl }}/blog/algorithmic/bloom-filters), and the two are cousins in the family of probabilistic data structures.

##1. The problem : counting distinct things is expensive
Counting *events* is trivial : keep one integer and increment it. Counting *distinct* events, the **cardinality** of a set, is the hard part, because to know whether the ID you just saw is new, you seemingly must remember every ID seen so far. Exact cardinality of $n$ distinct items needs $O(n)$ memory. HyperLogLog breaks that barrier.

<div class="definition">
	The <b>cardinality</b> of a stream is its number of <b>distinct</b> elements. HyperLogLog estimates it in a single pass using memory that stays essentially <b>constant</b>, a few kilobytes, no matter how many billions of items flow past.
</div>

##1. The coin-flip intuition
Here is the whole idea in a game. Ask a friend to flip a coin repeatedly and remember only the **longest run of heads** they ever saw at the start of a flip-sequence. If they report a run of $3$ heads, you can guess they did not flip many times ; if they report a run of $20$, they must have been flipping for ages, because a run of $20$ heads is astronomically rare ($\text{probability } 2^{-20}$).

HyperLogLog plays exactly this game with **hashes**. Each item is hashed to a random-looking binary string. We look at the position of the first $1$ bit, equivalently, the number of **leading zeros**. A hash beginning with $\rho$ zeros is as rare as flipping $\rho$ heads in a row (probability $2^{-\rho-1}$). So if, across the whole stream, the *most* leading zeros we ever saw is $\rho$, a good bet is that we have processed roughly $2^{\rho}$ distinct items.

<figure><center>
<svg viewBox="0 0 640 250" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g1-bit{ font-family:'Source Code Pro', monospace; font-size:17px; fill:#2b2b2b; }
    .g1-z{ fill:#b5651d; font-weight:bold; }
    .g1-item{ font-size:13px; fill:#7a7a7a; font-style:italic; }
    .g1-lab{ font-size:12px; fill:#7a7a7a; }
    .g1-reg{ fill:#f3faf4; stroke:#5aa06a; stroke-width:1.6; }
    .g1-rt{ font-size:14px; fill:#2b2b2b; text-anchor:middle; }
    .g1-pulse{ animation:pz 3s ease-in-out infinite; transform-box:fill-box; transform-origin:center; }
    @keyframes pz { 0%,100%{opacity:.6} 50%{opacity:1} }
  </style>
  <text class="g1-item" x="30" y="46">"alice@…"</text>
  <text class="g1-bit" x="150" y="50"><tspan class="g1-z">000</tspan>10110101101001</text>
  <text class="g1-lab" x="470" y="46">→ ρ = 3 leading zeros</text>
  <text class="g1-item" x="30" y="96">"bob@…"</text>
  <text class="g1-bit" x="150" y="100"><tspan class="g1-z">0</tspan>110100101100110</text>
  <text class="g1-lab" x="470" y="96">→ ρ = 1</text>
  <text class="g1-item" x="30" y="146">"carol@…"</text>
  <text class="g1-bit" x="150" y="150"><tspan class="g1-z">00000</tspan>101011010011</text>
  <text class="g1-lab" x="470" y="146">→ ρ = 5   (rare! )</text>
  <!-- max register -->
  <rect class="g1-reg g1-pulse" x="150" y="180" width="230" height="42" rx="8"/>
  <text class="g1-rt" x="265" y="207">max ρ so far  =  5   ⇒   ≈ 2⁵ = 32 distinct</text>
</svg>
<figcaption><b>Figure 1 -</b> Each item is hashed ; we count its leading zeros $\rho$. The <b>maximum</b> $\rho$ ever seen is a (very rough) exponent for the cardinality : the rarer the pattern we have stumbled on, the more items we must have drawn.</figcaption>
</center></figure>

##1. From a wild guess to a good estimate
A single "max leading zeros" is an absurdly noisy estimator, one lucky hash with many zeros throws it off by a factor of two. The fix is **stochastic averaging** : split the stream into $m$ independent groups (**registers**) using the first few bits of each hash to choose a group, and track the maximum $\rho$ **within each register** separately. Now we have $m$ noisy estimates instead of one, and we can average them.

The crucial invention of HyperLogLog is *how* to average [^1] : not the arithmetic mean, but the **harmonic mean** of the per-register estimates, which tames the outliers that a few large registers would otherwise cause. The cardinality estimate is

$$ \begin{equation} \hat{n} = \alpha_m\, m^{2} \left( \sum_{j=1}^{m} 2^{-M[j]} \right)^{-1}, \label{eq:hll} \end{equation} $$

where $M[j]$ is the maximum leading-zero count in register $j$ and $\alpha_m$ is a small bias-correction constant ($\alpha_m \approx 0.7213/(1 + 1.079/m)$).

<figure><center>
<svg viewBox="0 0 640 210" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g2-reg2{ fill:#f6f7fb; stroke:#4a6da7; stroke-width:1.4; }
    .g2-rv{ font-size:14px; fill:#2b2b2b; text-anchor:middle; } .g2-ri{ font-size:9.5px; fill:#9a9a9a; text-anchor:middle; }
    .g2-flow{ stroke:#b5651d; stroke-width:1.8; fill:none; stroke-dasharray:5 6; animation:hf 1.6s linear infinite; }
    @keyframes hf { to { stroke-dashoffset:-44; } }
    .g2-cap{ font-size:12px; fill:#7a7a7a; text-anchor:middle; } .g2-comb{ fill:#faf6f2; stroke:#b5651d; stroke-width:1.6; }
  </style>
  <text class="g2-cap" x="320" y="26">hash routes each item to one of m registers ; each keeps its own max ρ</text>
  <!-- stream in -->
  <path class="g2-flow" d="M20,70 L70,70"/>
  <g>
    <rect class="g2-reg2" x="72"  y="52" width="40" height="36" rx="5"/><text class="g2-rv" x="92"  y="76">2</text><text class="g2-ri" x="92"  y="100">M[0]</text>
    <rect class="g2-reg2" x="120" y="52" width="40" height="36" rx="5"/><text class="g2-rv" x="140" y="76">5</text><text class="g2-ri" x="140" y="100">M[1]</text>
    <rect class="g2-reg2" x="168" y="52" width="40" height="36" rx="5"/><text class="g2-rv" x="188" y="76">1</text><text class="g2-ri" x="188" y="100">M[2]</text>
    <rect class="g2-reg2" x="216" y="52" width="40" height="36" rx="5"/><text class="g2-rv" x="236" y="76">3</text><text class="g2-ri" x="236" y="100">M[3]</text>
    <rect class="g2-reg2" x="264" y="52" width="40" height="36" rx="5"/><text class="g2-rv" x="284" y="76">2</text><text class="g2-ri" x="284" y="100">M[4]</text>
    <rect class="g2-reg2" x="312" y="52" width="40" height="36" rx="5"/><text class="g2-rv" x="332" y="76">4</text><text class="g2-ri" x="332" y="100">M[5]</text>
    <rect class="g2-reg2" x="360" y="52" width="40" height="36" rx="5"/><text class="g2-rv" x="380" y="76">2</text><text class="g2-ri" x="380" y="100">…</text>
    <rect class="g2-reg2" x="408" y="52" width="40" height="36" rx="5"/><text class="g2-rv" x="428" y="76">3</text><text class="g2-ri" x="428" y="100">M[m-1]</text>
  </g>
  <path class="g2-flow" d="M456,70 L508,70"/>
  <rect class="g2-comb" x="510" y="46" width="118" height="48" rx="8"/><text class="g2-rv" x="569" y="66" font-size="12">harmonic</text><text class="g2-rv" x="569" y="82" font-size="12">mean → n̂</text>
  <text class="g2-cap" x="320" y="150">more registers m  ⇒  less noise · standard error ≈ 1.04 / √m</text>
  <text class="g2-cap" x="320" y="176">m = 16 384 registers × 6 bits ≈ 12 KB  →  counts billions within ~0.8%</text>
</svg>
<figcaption><b>Figure 2 -</b> Stochastic averaging. The hash sends each item to one of $m$ registers ; each stores only the largest leading-zero count it has seen. Combining them with the harmonic mean $\eqref{eq:hll}$ turns many noisy guesses into one accurate estimate.</figcaption>
</center></figure>

##1. Why it is astonishing
The accuracy of HyperLogLog is governed by the number of registers : the relative error is about

$$ \begin{equation} \frac{\sigma}{n} \approx \frac{1.04}{\sqrt{m}}. \label{eq:err} \end{equation} $$

*In plain words : quadrupling the number of registers halves the error.* Each register only needs to store a small integer (a count of leading zeros, at most $\sim 6$ bits). With $m = 16{,}384$ registers that is roughly **12 KB of memory**, and $\eqref{eq:err}$ gives an error under $1\%$, while counting cardinalities into the **billions**. An exact set would have needed gigabytes. That gap, kilobytes versus gigabytes, is why HyperLogLog is built into Redis (the `PFADD` / `PFCOUNT` commands), Presto, BigQuery, Redshift and countless analytics pipelines.

<div class="example">
	Two HyperLogLog sketches can be <b>merged</b> by taking, register by register, the larger of the two counts, this gives the sketch of the <i>union</i> of the two streams. So you can count today's unique visitors on each of $100$ servers independently and combine the sketches at the end to get the global unique count, exactly, with no re-processing. This effortless mergeability is a superpower for distributed systems.
</div>

##1. A family resemblance
It is worth pausing on how similar this is to [Bloom filters]({{ site.baseurl }}/blog/algorithmic/bloom-filters). Both start by hashing items into random-looking bits ; both keep a small fixed-size summary instead of the data ; both answer a question approximately, with a mathematically guaranteed error, that would otherwise cost linear memory. They differ in the question : a Bloom filter answers *"have I seen this exact item ?"* (membership), while HyperLogLog answers *"how many different items have I seen ?"* (cardinality). Together they are the two most famous members of a toolbox every large-scale engineer should know.

##1. Conclusion
HyperLogLog turns a childhood observation, that stumbling on a rare coincidence means you have tried many times, into a rigorous cardinality estimator. Hash each item, watch the leading zeros, spread the observations across many registers, and combine them with a harmonic mean : a few kilobytes then suffice to count billions of distinct things to within a percent, in one pass, and the sketches even merge for free. It is a small monument to how a clever dose of randomness can buy enormous savings, the same lesson its cousin the Bloom filter teaches, applied to a different question.


References
----------
{% bibliography --cited %}

[^1]: HyperLogLog is the last in a lineage : *Flajolet–Martin* (1985) introduced the leading-zeros idea, *LogLog* {% cite Durand2003 %} added registers, and HyperLogLog {% cite Flajolet2007 %} added the harmonic mean. Google's *HyperLogLog++* {% cite Heule2013 %} later refined the small-range bias and memory layout used in practice.
