---
layout: post
title: Introduction to Bloom Filters
date: 2023-01-18 12:00
comments: true
external-url:
categories: Algorithmic
tags: [python, algorithmic, probability]
permalink: /blog/:categories/bloom-filters
---


> Suppose you are building a web crawler and, for every URL you meet, you must answer one question : *"have I seen this before ?"*. Storing every URL in a hash set works, until the set grows to billions of entries and no longer fits in memory. What if you could answer that question using a few bits per item, accepting that you will occasionally be told *"yes"* when the true answer is *"no"*, but **never** the reverse ? This is exactly the bargain struck by the **Bloom filter**, a beautifully simple probabilistic data structure invented by Burton Bloom in 1970 {% cite Bloom1970 %}. In this post we build it from scratch, implement it in Python, and derive the mathematics that tells us how to size it.

##1. What is a Bloom filter ?
A Bloom filter is a probabilistic data structure for **approximate set membership**. It supports two operations, `insert(x)` and `contains(x)`, and it answers the second one with a deliberate asymmetry :

<div class="definition">
	A <b>Bloom filter</b> answers a membership query with either <i>"possibly in the set"</i> or <i>"definitely not in the set"</i>. It admits <b>false positives</b> (claiming an absent element is present) but never <b>false negatives</b> : if it says an element is absent, that is certain.
</div>

*In plain words : it is a forgetful bouncer with a perfect memory for "no". Ask it "is Alice on the guest list ?" and it either says "she might be, let me double-check" or "definitely not, turn her away", and when it says no, it is never wrong.*

That one-sided error is what buys the enormous space savings. A Bloom filter stores no keys at all, only a compact array of bits, and yet it can vouch, with a tunable and quantifiable error rate, that an element has never been inserted. When a definitive answer is occasionally required, the filter is used as a cheap first line of defence : only the queries it lets through need to touch the slow, exact store on disk.

##1. How it works
The machinery has three ingredients : a bit array $B$ of $m$ bits (all initially $0$), and $k$ independent hash functions $h_{1}, \dots, h_{k}$, each mapping an element to one of the $m$ positions.

**Insertion.** To insert an element $x$, compute the $k$ positions $h_{1}(x), \dots, h_{k}(x)$ and set each of those bits to $1$. Bits are only ever turned on, never off.

<figure><center>
<svg viewBox="0 0 640 240" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .cellb { fill:#fff; stroke:#c9cdd6; stroke-width:1.4; }
    .idx   { font-size:10.5px; fill:#9a9a9a; }
    .hpath { stroke:#b5651d; stroke-width:1.8; fill:none; opacity:.8;
             stroke-dasharray:4 6; animation:bflow 1.6s linear infinite; }
    @keyframes bflow { to { stroke-dashoffset:-40; } }
    .setb  { fill:#4a6da7; opacity:0; }
    .s0 { animation:lite 6s ease-in-out infinite; animation-delay:.3s; }
    .s1 { animation:lite 6s ease-in-out infinite; animation-delay:1.3s; }
    .s2 { animation:lite 6s ease-in-out infinite; animation-delay:2.3s; }
    @keyframes lite { 0%{opacity:0} 8%{opacity:1} 92%{opacity:1} 100%{opacity:0} }
    .elem { fill:#f3faf4; stroke:#5aa06a; stroke-width:1.6; }
    .lbl  { font-size:15px; fill:#2b2b2b; } .it{ font-style:italic; }
    .htag { font-size:12px; fill:#b5651d; }
  </style>
  <!-- element -->
  <rect class="elem" x="250" y="16" rx="16" ry="16" width="140" height="38"/>
  <text class="lbl it" x="320" y="41" text-anchor="middle">insert("apple")</text>
  <!-- hash tags -->
  <text class="htag" x="150" y="96" text-anchor="middle">h₁</text>
  <text class="htag" x="320" y="150" text-anchor="middle">h₂</text>
  <text class="htag" x="470" y="96" text-anchor="middle">h₃</text>
  <!-- hash paths to cells 3, 8, 12 -->
  <path class="hpath" d="M300,54 C230,90 175,110 140,150"/>
  <path class="hpath" d="M320,54 C320,120 320,120 320,150"/>
  <path class="hpath" d="M340,54 C420,90 470,110 500,150"/>
  <!-- bit array of 16 cells, width 32, start x=32, y=150 -->
  <g>
    <!-- generate cells -->
    <g>
      <rect class="cellb" x="32"  y="150" width="32" height="34"/><text class="idx" x="48"  y="200" text-anchor="middle">0</text>
      <rect class="cellb" x="64"  y="150" width="32" height="34"/><text class="idx" x="80"  y="200" text-anchor="middle">1</text>
      <rect class="cellb" x="96"  y="150" width="32" height="34"/><text class="idx" x="112" y="200" text-anchor="middle">2</text>
      <rect class="cellb" x="128" y="150" width="32" height="34"/><rect class="setb s0" x="128" y="150" width="32" height="34"/><text class="idx" x="144" y="200" text-anchor="middle">3</text>
      <rect class="cellb" x="160" y="150" width="32" height="34"/><text class="idx" x="176" y="200" text-anchor="middle">4</text>
      <rect class="cellb" x="192" y="150" width="32" height="34"/><text class="idx" x="208" y="200" text-anchor="middle">5</text>
      <rect class="cellb" x="224" y="150" width="32" height="34"/><text class="idx" x="240" y="200" text-anchor="middle">6</text>
      <rect class="cellb" x="256" y="150" width="32" height="34"/><text class="idx" x="272" y="200" text-anchor="middle">7</text>
      <rect class="cellb" x="288" y="150" width="32" height="34"/><rect class="setb s1" x="288" y="150" width="32" height="34"/><text class="idx" x="304" y="200" text-anchor="middle">8</text>
      <rect class="cellb" x="320" y="150" width="32" height="34"/><text class="idx" x="336" y="200" text-anchor="middle">9</text>
      <rect class="cellb" x="352" y="150" width="32" height="34"/><text class="idx" x="368" y="200" text-anchor="middle">10</text>
      <rect class="cellb" x="384" y="150" width="32" height="34"/><text class="idx" x="400" y="200" text-anchor="middle">11</text>
      <rect class="cellb" x="416" y="150" width="32" height="34"/><rect class="setb s2" x="416" y="150" width="32" height="34"/><text class="idx" x="432" y="200" text-anchor="middle">12</text>
      <rect class="cellb" x="448" y="150" width="32" height="34"/><text class="idx" x="464" y="200" text-anchor="middle">13</text>
      <rect class="cellb" x="480" y="150" width="32" height="34"/><text class="idx" x="496" y="200" text-anchor="middle">14</text>
      <rect class="cellb" x="512" y="150" width="32" height="34"/><text class="idx" x="528" y="200" text-anchor="middle">15</text>
    </g>
  </g>
  <text class="lbl" x="288" y="228" text-anchor="middle" font-size="13" fill="#7a7a7a">the bit array B  (m = 16 bits, k = 3 hashes)</text>
</svg>
<figcaption><b>Figure 1 -</b> Inserting <i>"apple"</i>. The three hash functions select positions $3, 8, 12$, and those bits are switched on (blue). Different elements light up different, possibly overlapping, positions.</figcaption>
</center></figure>

**Membership query.** To test whether $x$ is present, compute the same $k$ positions and inspect the bits. If **any** of them is $0$, then $x$ was certainly never inserted, because insertion would have set all of them, this is the guaranteed *no false negatives* property. If **all** $k$ bits are $1$, the filter reports *"possibly present"* : the element may have been inserted, or those bits may simply have been set by a combination of other elements, a **false positive**.

<figure><center>
<svg viewBox="0 0 640 250" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .c0 { fill:#fff; stroke:#c9cdd6; stroke-width:1.3; }
    .c1 { fill:#4a6da7; stroke:#3c5a8a; stroke-width:1.3; }
    .bit{ font-size:13px; fill:#fff; font-weight:bold; }
    .bit0{ font-size:13px; fill:#b9bdc6; }
    .qlbl{ font-size:14px; fill:#2b2b2b; font-style:italic; }
    .yes { font-size:14px; fill:#5aa06a; font-weight:bold; }
    .no  { font-size:14px; fill:#b5651d; font-weight:bold; }
    .mk  { font-size:11px; fill:#7a7a7a; }
  </style>
  <!-- row A: member -->
  <text class="qlbl" x="14" y="55">contains("apple")</text>
  <g transform="translate(190,32)">
    <rect class="c1" x="0"   y="0" width="34" height="34"/><text class="bit" x="17"  y="22" text-anchor="middle">1</text>
    <rect class="c1" x="40"  y="0" width="34" height="34"/><text class="bit" x="57"  y="22" text-anchor="middle">1</text>
    <rect class="c1" x="80"  y="0" width="34" height="34"/><text class="bit" x="97"  y="22" text-anchor="middle">1</text>
    <text class="mk" x="17" y="52" text-anchor="middle">pos 3</text>
    <text class="mk" x="57" y="52" text-anchor="middle">pos 8</text>
    <text class="mk" x="97" y="52" text-anchor="middle">pos 12</text>
  </g>
  <text class="yes" x="360" y="55">all 1  →  possibly in set ✓</text>
  <!-- divider -->
  <line x1="14" y1="120" x2="626" y2="120" stroke="#eee" stroke-width="1"/>
  <!-- row B: non-member -->
  <text class="qlbl" x="14" y="175">contains("cow")</text>
  <g transform="translate(190,152)">
    <rect class="c1" x="0"   y="0" width="34" height="34"/><text class="bit" x="17"  y="22" text-anchor="middle">1</text>
    <rect class="c0" x="40"  y="0" width="34" height="34"/><text class="bit0" x="57" y="22" text-anchor="middle">0</text>
    <rect class="c1" x="80"  y="0" width="34" height="34"/><text class="bit" x="97"  y="22" text-anchor="middle">1</text>
    <text class="mk" x="17" y="52" text-anchor="middle">pos 5</text>
    <text class="mk" x="57" y="52" text-anchor="middle">pos 9</text>
    <text class="mk" x="97" y="52" text-anchor="middle">pos 12</text>
  </g>
  <text class="no" x="360" y="175">a 0  →  definitely not in set ✗</text>
</svg>
<figcaption><b>Figure 2 -</b> The asymmetric query logic. A single zero bit is an irrefutable proof of absence ; all-ones is only circumstantial evidence of presence.</figcaption>
</center></figure>

##1. A Python implementation
Real Bloom filters do not need $k$ genuinely independent hash functions. A classic result by Kirsch and Mitzenmacher {% cite Kirsch2006 %} shows that two hash functions are enough : the family $h_{i}(x) = h_{1}(x) + i \cdot h_{2}(x) \pmod{m}$ behaves, for our purposes, just like $k$ independent ones. We use Python's `hashlib` to derive $h_{1}$ and $h_{2}$ from a single digest.

```python
import hashlib
from math import log

class BloomFilter:
    def __init__(self, m: int, k: int):
        self.m = m                     # number of bits
        self.k = k                     # number of hash functions
        self.bits = bytearray((m + 7) // 8)

    def _positions(self, item: str):
        digest = hashlib.sha256(item.encode()).digest()
        h1 = int.from_bytes(digest[:8],  "big")
        h2 = int.from_bytes(digest[8:16], "big")
        for i in range(self.k):
            yield (h1 + i * h2) % self.m          # Kirsch-Mitzenmacher

    def add(self, item: str):
        for pos in self._positions(item):
            self.bits[pos // 8] |= 1 << (pos % 8)

    def __contains__(self, item: str) -> bool:
        return all(
            self.bits[pos // 8] & (1 << (pos % 8))
            for pos in self._positions(item)
        )
```

The usage mirrors the guarantee exactly, a negative answer is always trustworthy :

```python
bf = BloomFilter(m=1000, k=7)
for word in ["apple", "banana", "cherry"]:
    bf.add(word)

print("apple"  in bf)   # True  (really inserted)
print("cow"    in bf)   # False (guaranteed correct)
print("grape"  in bf)   # False, or rarely True (a false positive)
```

##1. The mathematics of false positives
The whole design hinges on one number : the probability that a query for an element we never inserted comes back positive. Let us derive it.

Assume the hash functions distribute elements uniformly and independently over the $m$ bits. When we insert a single element we set $k$ bits ; the probability that one *specific* bit is left untouched by one hash is $1 - \tfrac{1}{m}$. After inserting $n$ elements, each contributing $k$ hashes, the probability that a given bit is still $0$ is

$$ \begin{equation} P(\text{bit} = 0) = \left(1 - \frac{1}{m}\right)^{kn} \approx e^{-kn/m}, \label{eq:bitzero} \end{equation} $$

using the standard limit $\left(1 - \tfrac{1}{m}\right)^{m} \to e^{-1}$. Consequently the probability that a given bit is $1$ is $\;1 - e^{-kn/m}$.

A false positive occurs when a *non-inserted* element hashes to $k$ positions that all happen to be $1$. Treating those $k$ bits as independent [^1], the false-positive probability is

$$ \begin{equation} \varepsilon \;\approx\; \left( 1 - e^{-kn/m} \right)^{k}. \label{eq:fp} \end{equation} $$

This compact formula is the design equation of every Bloom filter. *In plain words : the more elements you cram in (larger $n$) the more crowded with $1$s the array gets, so accidental all-ones collisions, false positives, become more likely ; give it more room (larger $m$) and they become rarer.*

##1. Choosing the parameters
Formula $\eqref{eq:fp}$ exposes a genuine trade-off in $k$. Too few hash functions and each query inspects too little evidence ; too many and the array saturates with $1$s, so almost everything looks present. There is a sweet spot, found by minimising $\varepsilon$ over $k$ for fixed $m$ and $n$ :

$$ \begin{equation} k^{\ast} = \frac{m}{n}\,\ln 2 \;\approx\; 0.693\,\frac{m}{n}. \label{eq:koptimal} \end{equation} $$

<figure><center>
<svg viewBox="0 0 640 350" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .axis{ stroke:#8a8a8a; stroke-width:1.3; }
    .grid{ stroke:#eee; stroke-width:1; }
    .curve{ stroke:#4a6da7; stroke-width:2.4; fill:none; }
    .tick{ font-size:11px; fill:#7a7a7a; }
    .axl { font-size:13px; fill:#2b2b2b; }
    .opt { stroke:#b5651d; stroke-width:1.4; stroke-dasharray:4 4; }
    .optd{ fill:#b5651d; }
    .optt{ font-size:12px; fill:#b5651d; }
  </style>
  <!-- gridlines -->
  <line class="grid" x1="70" y1="60"  x2="600" y2="60"/>
  <line class="grid" x1="70" y1="180" x2="600" y2="180"/>
  <!-- axes -->
  <line class="axis" x1="70" y1="60" x2="70" y2="300"/>
  <line class="axis" x1="70" y1="300" x2="600" y2="300"/>
  <!-- y ticks -->
  <text class="tick" x="62" y="304" text-anchor="end">0</text>
  <text class="tick" x="62" y="184" text-anchor="end">0.05</text>
  <text class="tick" x="62" y="64"  text-anchor="end">0.10</text>
  <text class="axl" x="24" y="185" text-anchor="middle" transform="rotate(-90 24 185)">false-positive rate ε</text>
  <!-- x ticks (k = 1..14), x = 90 + (k-1)*38.46 -->
  <text class="tick" x="90"  y="318" text-anchor="middle">1</text>
  <text class="tick" x="205" y="318" text-anchor="middle">4</text>
  <text class="tick" x="321" y="318" text-anchor="middle">7</text>
  <text class="tick" x="436" y="318" text-anchor="middle">10</text>
  <text class="tick" x="590" y="318" text-anchor="middle">14</text>
  <text class="axl" x="335" y="340" text-anchor="middle">number of hash functions k   (with m/n = 10)</text>
  <!-- curve through computed points of (1 - e^{-k/10})^k -->
  <path class="curve" d="M90,71 L128,221 L167,258 L205,272 L244,277 L282,280 L321,280 L359,280 L398,278 L436,275 L475,270 L513,261 L551,247 L590,228"/>
  <!-- optimum marker at k*=6.93 -> x ~ 318 -->
  <line class="opt" x1="318" y1="280" x2="318" y2="60"/>
  <circle class="optd" cx="321" cy="280" r="4"/>
  <text class="optt" x="326" y="52">k* = (m/n) ln 2 ≈ 6.9</text>
</svg>
<figcaption><b>Figure 3 -</b> False-positive rate $\varepsilon$ from $\eqref{eq:fp}$ as a function of $k$, for a filter with ten bits per element ($m/n = 10$). The curve bottoms out near $k^{\ast} \approx 6.9$, matching $\eqref{eq:koptimal}$ ; beyond it, extra hashes only saturate the array and hurt.</figcaption>
</center></figure>

Substituting $k^{\ast}$ back into $\eqref{eq:fp}$ and rearranging gives the number of bits required to hit a target error rate $\varepsilon$ :

$$ \begin{equation} m = -\,\frac{n \ln \varepsilon}{(\ln 2)^{2}}. \label{eq:msize} \end{equation} $$

<div class="example">
	Say we expect $n = 1{,}000{,}000$ elements and want a false-positive rate of $\varepsilon = 1\%$. Then $\eqref{eq:msize}$ gives $m \approx 9.585\,n \approx 9.59$ million bits, about $1.2$ MB, and $\eqref{eq:koptimal}$ suggests $k^{\ast} \approx 7$ hash functions. Roughly <b>ten bits per element</b> buys a $1\%$ error rate ; note this is entirely independent of how <i>large</i> the elements themselves are, be they short words or kilobyte-long URLs.
</div>

##1. Variants and where they are used
The plain Bloom filter cannot delete elements (turning a bit off might erase evidence of some *other* element). This limitation spawned a family of refinements :

- **Counting Bloom filters** replace each bit by a small counter, so insertions increment and deletions decrement, restoring removal at the cost of more space {% cite Fan2000 %}.
- **Scalable** and **partitioned** Bloom filters grow gracefully when $n$ is not known in advance.
- **Cuckoo filters** achieve similar space efficiency while supporting deletion and often lower false-positive rates.

Bloom filters are quietly everywhere. Databases such as Cassandra, HBase and Google's Bigtable use them to avoid disk reads for keys that are absent ; the same trick guards the LSM-tree lookups in RocksDB. Web browsers have used them for safe-browsing URL blacklists, content-delivery networks use them for cache filtering, and Bitcoin's lightweight clients once used them to request relevant transactions without revealing exactly which addresses they cared about. Whenever *"probably yes / definitely no"* is good enough, a Bloom filter turns a memory problem into a handful of bits.

##1. Conclusion
The Bloom filter is a small masterpiece of engineering trade-offs : by giving up the ability to store keys, and by tolerating a controlled, quantifiable rate of false positives, it answers set-membership queries in constant time and a fistful of bits per element. Its one-sided error, *never* a false negative, is what makes it trustworthy where it matters, and its design equations $\eqref{eq:fp}$–$\eqref{eq:msize}$ let us dial the error rate to whatever a system can afford. More than fifty years after Burton Bloom described it, it remains a first reflex for anyone who needs to check things without the luxury of remembering them.


References
----------
{% bibliography --cited %}

[^1]: This independence assumption is a slight simplification, the events "bit $i$ is set" are weakly correlated, so $\eqref{eq:fp}$ is a very good approximation rather than an exact identity. Bose *et al.* derived the exact expression, which differs only negligibly for the array sizes used in practice.
