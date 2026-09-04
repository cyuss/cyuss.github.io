---
layout: post
title: Kolmogorov's Superposition Theorem
date: 2021-06-15 12:00
comments: true
external-url:
categories: Mathematics
tags: [analysis, machine_learning]
permalink: /blog/:categories/kolmogorov-superposition
---


> At the 1900 International Congress of Mathematicians, David Hilbert presented a list of 23 problems that would shape the century to come. His **thirteenth** asked, in essence, a deceptively simple question : are there functions of several variables that cannot be built from functions of fewer variables ? Half a century later, in a two-page note, Andrey Kolmogorov gave a breathtaking answer : **every** continuous function of many variables is a superposition of continuous functions of a *single* variable and addition. Nothing more than univariate functions and $+$ is ever needed. This post tells that story, unpacks the structure of the theorem, and follows the surprising thread that ties it to modern neural networks.

##1. Hilbert's thirteenth problem
Some functions of two variables are obviously reducible. The product $xy$, for instance, hides a univariate skeleton :

$$ \begin{equation} xy = \exp\big( \ln x + \ln y \big), \qquad x, y > 0, \label{eq:product} \end{equation} $$

so multiplication is really just addition of logarithms wrapped inside an exponential, *i.e.* a superposition of the univariate functions $\ln$ and $\exp$. Hilbert asked whether this kind of reduction is always possible. Concretely, he conjectured that the solution of the general seventh-degree equation, viewed as a function of three of its coefficients, could **not** be written using only continuous functions of two variables. He expected an *impossibility* result.

Mathematics rarely obliges our intuitions. In 1957 Kolmogorov {% cite Kolmogorov1957 %}, building on and then dramatically simplifying work with his student Vladimir Arnold {% cite Arnold1957 %} [^1], proved the opposite of what Hilbert anticipated : not only are two variables enough, a *single* variable is enough, provided we are allowed to add.

##1. The theorem
<div class="theorem" text="Kolmogorov, 1957">
	For every integer $n \geq 2$ there exist continuous functions $\phi_{q,p} : [0,1] \rightarrow \Bbb{R}$, for $q = 0, \dots, 2n$ and $p = 1, \dots, n$, such that every continuous function $ƒ : [0,1]^{n} \rightarrow \Bbb{R}$ can be represented in the form
	$$ \begin{equation} ƒ(x_{1}, \dots, x_{n}) = \sum_{q=0}^{2n} \Phi_{q}\!\left( \sum_{p=1}^{n} \phi_{q,p}(x_{p}) \right), \label{eq:kolmogorov} \end{equation} $$
	where the outer functions $\Phi_{q} : \Bbb{R} \rightarrow \Bbb{R}$ are continuous and depend on $ƒ$.
</div>

Read this slowly, because every symbol is carrying weight. The **inner functions** $\phi_{q,p}$ act on a single coordinate $x_{p}$ at a time. Their outputs are summed, coordinate by coordinate, into $2n+1$ scalars $z_{q} = \sum_{p} \phi_{q,p}(x_{p})$. Each scalar is then reshaped by a single **outer function** $\Phi_{q}$, and the $2n+1$ results are added. The entire multivariate complexity of $ƒ$ has been squeezed into univariate maps and two rounds of addition.

*In plain words : no matter how complicated a recipe is, Kolmogorov says you can cook it using only single-ingredient preparations and the act of piling things together. Each $\phi_{q,p}$ processes one "ingredient" (one coordinate) on its own ; you add up the results, run them through the $\Phi_q$, and add again. Nothing ever has to combine two inputs in a genuinely two-dimensional way.*

The structure is easiest to see as a graph. The following diagram traces the flow for $n = 2$ : two inputs, $2n + 1 = 5$ inner units, five outer units, one sum.

<figure><center>
<svg viewBox="0 0 720 415" width="100%" style="max-width:640px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g1-edge { stroke:#9db2d8; stroke-width:1.4; fill:none; opacity:.55;
            stroke-dasharray:5 9; animation:kflow 2.6s linear infinite; }
    .g1-edge2{ stroke:#d8b89d; stroke-width:1.6; fill:none; opacity:.75;
            stroke-dasharray:5 9; animation:kflow 2.6s linear infinite; }
    @keyframes kflow { to { stroke-dashoffset:-56; } }
    .g1-lbl { font-size:16px; fill:#2b2b2b; } .g1-lblf{ font-style:italic; }
    .g1-sub { font-size:11px; }
    .g1-cap { font-size:12.5px; fill:#7a7a7a; }
    .g1-nin { fill:#f6f7fb; stroke:#4a6da7; stroke-width:1.6; }
    .g1-ninr{ fill:#f3faf4; stroke:#5aa06a; stroke-width:1.6; }
    .g1-nout{ fill:#faf6f2; stroke:#b5651d; stroke-width:1.6; }
  </style>
  <!-- column captions -->
  <text class="g1-cap" x="70"  y="26" text-anchor="middle">inputs</text>
  <text class="g1-cap" x="270" y="26" text-anchor="middle">inner (φ)</text>
  <text class="g1-cap" x="470" y="26" text-anchor="middle">outer (Φ)</text>
  <text class="g1-cap" x="650" y="26" text-anchor="middle">output</text>
  <!-- edges inputs -> inner -->
  <g>
    <path class="g1-edge" d="M92,150 L248,70"/>  <path class="g1-edge" d="M92,150 L248,145"/>
    <path class="g1-edge" d="M92,150 L248,215"/> <path class="g1-edge" d="M92,150 L248,285"/>
    <path class="g1-edge" d="M92,150 L248,360"/>
    <path class="g1-edge" d="M92,280 L248,70"/>  <path class="g1-edge" d="M92,280 L248,145"/>
    <path class="g1-edge" d="M92,280 L248,215"/> <path class="g1-edge" d="M92,280 L248,285"/>
    <path class="g1-edge" d="M92,280 L248,360"/>
  </g>
  <!-- edges inner -> outer (one-to-one) -->
  <g>
    <path class="g1-edge2" d="M292,70  L448,70"/>  <path class="g1-edge2" d="M292,145 L448,145"/>
    <path class="g1-edge2" d="M292,215 L448,215"/> <path class="g1-edge2" d="M292,285 L448,285"/>
    <path class="g1-edge2" d="M292,360 L448,360"/>
  </g>
  <!-- edges outer -> sum -->
  <g>
    <path class="g1-edge2" d="M492,70  L628,215"/> <path class="g1-edge2" d="M492,145 L628,215"/>
    <path class="g1-edge2" d="M492,215 L628,215"/> <path class="g1-edge2" d="M492,285 L628,215"/>
    <path class="g1-edge2" d="M492,360 L628,215"/>
  </g>
  <!-- input nodes -->
  <circle class="g1-nin" cx="70" cy="150" r="22"/><text class="g1-lbl g1-lblf" x="70" y="156" text-anchor="middle">x<tspan class="g1-sub" dy="4">1</tspan></text>
  <circle class="g1-nin" cx="70" cy="280" r="22"/><text class="g1-lbl g1-lblf" x="70" y="286" text-anchor="middle">x<tspan class="g1-sub" dy="4">2</tspan></text>
  <!-- inner nodes: z_q -->
  <g class="g1-lbl g1-lblf" text-anchor="middle">
    <circle class="g1-ninr" cx="270" cy="70"  r="20"/><text x="270" y="76">z<tspan class="g1-sub" dy="4">0</tspan></text>
    <circle class="g1-ninr" cx="270" cy="145" r="20"/><text x="270" y="151">z<tspan class="g1-sub" dy="4">1</tspan></text>
    <circle class="g1-ninr" cx="270" cy="215" r="20"/><text x="270" y="221">z<tspan class="g1-sub" dy="4">2</tspan></text>
    <circle class="g1-ninr" cx="270" cy="285" r="20"/><text x="270" y="291">z<tspan class="g1-sub" dy="4">3</tspan></text>
    <circle class="g1-ninr" cx="270" cy="360" r="20"/><text x="270" y="366">z<tspan class="g1-sub" dy="4">4</tspan></text>
  </g>
  <!-- outer nodes: Phi_q -->
  <g class="g1-lbl" text-anchor="middle">
    <circle class="g1-nout" cx="470" cy="70"  r="20"/><text x="470" y="76">Φ<tspan class="g1-sub" dy="4">0</tspan></text>
    <circle class="g1-nout" cx="470" cy="145" r="20"/><text x="470" y="151">Φ<tspan class="g1-sub" dy="4">1</tspan></text>
    <circle class="g1-nout" cx="470" cy="215" r="20"/><text x="470" y="221">Φ<tspan class="g1-sub" dy="4">2</tspan></text>
    <circle class="g1-nout" cx="470" cy="285" r="20"/><text x="470" y="291">Φ<tspan class="g1-sub" dy="4">3</tspan></text>
    <circle class="g1-nout" cx="470" cy="360" r="20"/><text x="470" y="366">Φ<tspan class="g1-sub" dy="4">4</tspan></text>
  </g>
  <!-- sum node -->
  <circle cx="650" cy="215" r="26" fill="#fff" stroke="#2b2b2b" stroke-width="1.8"/>
  <text class="g1-lbl" x="650" y="224" text-anchor="middle" font-size="24">Σ</text>
  <text class="g1-lbl g1-lblf" x="650" y="272" text-anchor="middle">ƒ(x<tspan class="g1-sub" dy="4">1</tspan><tspan dy="-4">,x</tspan><tspan class="g1-sub" dy="4">2</tspan><tspan dy="-4">)</tspan></text>
</svg>
<figcaption><b>Figure 1 -</b> Kolmogorov's representation for $n=2$. Each input passes through inner univariate maps $\phi_{q,p}$ (green), the coordinate contributions are summed into $z_q$, reshaped by outer maps $\Phi_q$ (orange), and finally added. The animated dashes suggest the direction of the computation.</figcaption>
</center></figure>

If this picture reminds you of something, that is not an accident, and we will return to it in the last section.

##1. What makes it remarkable
Three features of $\eqref{eq:kolmogorov}$ deserve to be highlighted, because each one defies a natural expectation.

<div class="definition">
	We call the $\phi_{q,p}$ the <b>inner</b> (or <i>universal</i>) functions and the $\Phi_{q}$ the <b>outer</b> functions. A representation is said to be <i>universal</i> when the inner functions can be chosen <b>once and for all</b>, independently of the target $ƒ$, so that only the outer functions have to adapt.
</div>

- **A fixed number of terms.** The outer sum always has exactly $2n + 1$ terms, no matter how wild $ƒ$ is. The complexity of $ƒ$ is absorbed entirely by the *shape* of the univariate functions, never by their *number*.

- **Universality of the inner functions.** In the sharper forms due to Lorentz, Sprecher and others {% cite Sprecher1965 %}, the inner functions $\phi_{q,p}$ (and even a single monotone $\phi$ with rational shifts) can be fixed *before* $ƒ$ is known. They form a kind of universal coordinate system into which any continuous function can be poured ; only the $\Phi_{q}$ then encode $ƒ$.

- **Continuity without smoothness.** The functions produced by the theorem are continuous but, in general, extremely irregular : they are typically nowhere differentiable, fractal-like objects. Vitushkin later showed that if one insists on *smoothness* (continuous derivatives), the reduction genuinely fails, which is why Hilbert's intuition was not entirely misplaced, it was merely aimed at the wrong regularity class.

<div class="example">
	The exponential trick $\eqref{eq:product}$ is a baby version of the phenomenon : the two-variable function $xy$ is expressed through the univariate $\ln$ and $\exp$ and one addition. Kolmogorov's theorem asserts that a decomposition of this flavour, though far less explicit, exists for <i>every</i> continuous $ƒ$ on the cube, not just for the analytically lucky ones.
</div>

##1. A non-constructive miracle
It is tempting to ask for the functions. Here the theorem turns coy : its original proof is an *existence* argument. It guarantees that suitable $\phi_{q,p}$ and $\Phi_{q}$ exist, but the classical constructions produce functions of formidable irregularity, defined as limits of carefully nested approximations. Explicit and numerically usable constructions of the inner functions were only obtained much later, notably by Sprecher and, more recently, through the algorithm of Braun and Griebel {% cite Braun2009 %}, which finally made the decomposition computable to a prescribed accuracy.

So the theorem is best read as a statement about *what is possible in principle*. It tells us that the barrier Hilbert imagined, dimensionality as an irreducible source of complexity, simply is not there for continuous functions. Whether that possibility can be *exploited* is a separate, and much harder, story, a tension we already met with the [Universal Approximation Theorem]({{ site.baseurl }}/blog/mathematics/universal-approximation-theorem).

##1. The neural-network connection
Look again at Figure 1. Inputs feed a first layer of univariate transformations, the results are summed, a second layer of univariate transformations is applied, and everything is added. That is precisely the shape of a **two-layer feedforward network** : the inner functions play the role of the first hidden layer, the outer functions the role of the second.

This resemblance was made explicit by Hecht-Nielsen {% cite HechtNielsen1987 %}, who observed that Kolmogorov's theorem can be phrased as the existence of an exact four-layer "Kolmogorov network" computing any continuous $ƒ$. For a while this was advertised as a theoretical foundation for neural networks : here, it seemed, was a proof that networks can represent *anything*, and with a fixed, tiny width of $2n+1$ units.

The enthusiasm needed tempering. Girosi and Poggio {% cite Girosi1989 %} pointed out the catch : the outer functions $\Phi_{q}$ depend on $ƒ$ in a wholly non-constructive and highly non-smooth way, so they are nothing like the fixed, simple activation functions (a sigmoid, a ReLU) used in real networks. Kolmogorov's theorem hands you an *exact representation* but with pathological building blocks ; the Universal Approximation Theorem hands you an *approximation* with benign, tunable building blocks. They are complementary lenses on the same question, how much can univariate non-linearities and addition achieve, and neither one, on its own, explains why deep learning works in practice.

##1. Conclusion
Kolmogorov's superposition theorem is one of those results that quietly rearranges your sense of what is possible. A question that Hilbert framed as a search for irreducible multivariate complexity turned out to have the most economical answer imaginable : univariate functions and addition suffice, always, for continuous functions on a cube. The price is regularity, the ingredients are fractal, non-constructive, and unfriendly to computation, which is exactly why the theorem informs, but does not by itself explain, the practical success of the layered architectures it so eerily prefigures. It is a bridge between a 1900 problem list and a 21st-century machine, built entirely out of one-dimensional pieces.


References
----------
{% bibliography --cited %}

[^1]: Arnold first settled the three-variable case of Hilbert's problem in 1957 ; Kolmogorov's general superposition theorem, of which this post treats, appeared the same year and subsumed it.
