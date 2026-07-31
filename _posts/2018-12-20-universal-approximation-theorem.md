---
layout: post
title: The Universal Approximation Theorem
date: 2021-04-16 12:00
comments: true
external-url:
categories: Mathematics
tags: [algorithmic, machine_learning]
permalink: /blog/:categories/universal-approximation-theorem
---


> Why do neural networks work at all ? Before worrying about *how* to train a network, it is worth asking whether a network is even *capable* of representing the function we are looking for. The **Universal Approximation Theorem** answers exactly this question : a feedforward network with a single hidden layer can approximate, as closely as we want, any continuous function on a compact domain. It is one of the most cited and, ironically, one of the most misunderstood results in machine learning. In this post we state it carefully, sketch why it is true, and, just as importantly, discuss what it does **not** say.

<!--## Notations

$$\begin{array}{ r c l }
\Bbb{R}^{n} & : & \text{the } n \text{-dimensional Euclidean space} \\
I_n & : & [0, 1]^n \text{, the unit hypercube} \\
C(I_n) & : & \text{the space of continuous functions on } I_n \\
\|\,.\,\|_{\infty} & : & \text{the supremum (uniform) norm} \\
M(I_n) & : & \text{the space of finite signed regular Borel measures on } I_n \\
\end{array}$$-->

##1. Introduction
A single artificial neuron computes a very simple thing : it takes an input vector $x \in \Bbb{R}^{n}$, forms a weighted sum $w^{T} x + b$ with weights $w \in \Bbb{R}^{n}$ and a bias $b \in \Bbb{R}$, and passes the result through a non-linear **activation function** $\sigma : \Bbb{R} \rightarrow \Bbb{R}$. A network with one hidden layer of $N$ such neurons and a linear output is therefore a function of the form

$$ \begin{equation} G(x) = \sum_{j=1}^{N} \alpha_{j} \, \sigma(w_{j}^{T} x + b_{j}) \label{eq:network} \end{equation} $$

where $\alpha_{j}, b_{j} \in \Bbb{R}$ and $w_{j} \in \Bbb{R}^{n}$. The set of all functions of the form $\eqref{eq:network}$, for every possible width $N$ and every choice of parameters, is exactly the class of functions that a shallow network can *represent*.

The question is then very natural. Given a target function $ƒ$ that we would like the network to compute, and a tolerance $\varepsilon > 0$, does there always exist a choice of parameters such that $G$ is $\varepsilon$-close to $ƒ$ everywhere ? The Universal Approximation Theorem says **yes**, provided $ƒ$ is continuous, the domain is compact, and $\sigma$ is chosen sensibly. What is striking is that a *single* hidden layer already suffices ; depth is not required for representability.

<div class="definition">
	A function $\sigma : \Bbb{R} \rightarrow \Bbb{R}$ is said to be <b>sigmoidal</b> if it is bounded and satisfies
	$$ \sigma(t) \rightarrow 1 \; \text{ as } \; t \rightarrow +\infty \qquad \text{and} \qquad \sigma(t) \rightarrow 0 \; \text{ as } \; t \rightarrow -\infty. $$
	The <a href="https://en.wikipedia.org/wiki/Logistic_function">logistic function</a> $\sigma(t) = \frac{1}{1 + e^{-t}}$ is the canonical example.
</div>

##1. Measuring the approximation
To say that $G$ is *close* to $ƒ$ we need a notion of distance between functions. We work on the unit hypercube $I_{n} = [0, 1]^{n}$, which is compact [^1], and on the space $C(I_{n})$ of continuous real functions on $I_{n}$, equipped with the **supremum norm**

$$ \begin{equation} \|ƒ - G\|_{\infty} = \sup_{x \in I_{n}} |ƒ(x) - G(x)|. \label{eq:supnorm} \end{equation} $$

Saying that networks are *universal approximators* is the topological statement that the family $\eqref{eq:network}$ is **dense** in $C(I_{n})$ : every continuous $ƒ$ is a limit, in the norm $\eqref{eq:supnorm}$, of functions computable by a one hidden layer network. Concretely, for every $\varepsilon > 0$ there is a network $G$ such that $\|ƒ - G\|_{\infty} < \varepsilon$.

##1. The theorem
The cleanest and most famous formulation is due to Cybenko {% cite Cybenko1989 %}, who proved it for continuous sigmoidal activations. The key technical notion is that of a *discriminatory* function.

<div class="definition">
	Let $M(I_{n})$ be the space of finite signed regular Borel measures on $I_{n}$. A function $\sigma$ is called <b>discriminatory</b> if the only measure $\mu \in M(I_{n})$ satisfying
	$$ \int_{I_{n}} \sigma(w^{T} x + b) \, d\mu(x) = 0 \qquad \text{for all } w \in \Bbb{R}^{n}, \, b \in \Bbb{R} $$
	is the zero measure $\mu = 0$.
</div>

Intuitively, a function is discriminatory if the ridge functions $x \mapsto \sigma(w^{T}x + b)$ are *rich enough* that no non-trivial measure can be orthogonal to all of them at once. With this vocabulary the theorem is remarkably compact.

<div class="theorem" text="Cybenko, 1989">
	Let $\sigma$ be any continuous discriminatory function. Then finite sums of the form
	$$ G(x) = \sum_{j=1}^{N} \alpha_{j} \, \sigma(w_{j}^{T} x + b_{j}) $$
	are <b>dense</b> in $C(I_{n})$. That is, for every $ƒ \in C(I_{n})$ and every $\varepsilon > 0$, there exists such a $G$ with $\;\|ƒ - G\|_{\infty} < \varepsilon$.
</div>

On its own this would be an abstract statement about a mysterious class of functions, were it not for the following companion lemma, which is what makes the theorem usable in practice.

<div class="theorem" text="Cybenko, 1989">
	Every bounded, measurable sigmoidal function is discriminatory. In particular, every <b>continuous</b> sigmoidal function is discriminatory.
</div>

Combining the two, any continuous sigmoidal activation, the logistic function, $\tanh$, the arctangent, yields a universal approximator. This is the result usually quoted simply as *the* Universal Approximation Theorem.

##1. Why is it true ?
The proof is short but leans on two pillars of functional analysis, and understanding the *shape* of the argument is more illuminating than the technical details.

Let $S \subset C(I_{n})$ be the linear subspace of all functions of the form $\eqref{eq:network}$. We want to show that its closure $\overline{S}$ is all of $C(I_{n})$. Suppose, for contradiction, that it is not : $\overline{S}$ is then a *proper* closed subspace.

- By the <a href="https://en.wikipedia.org/wiki/Hahn%E2%80%93Banach_theorem">Hahn–Banach theorem</a>, there exists a non-zero bounded linear functional $L$ on $C(I_{n})$ that vanishes on the whole of $\overline{S}$, hence on $S$.
- By the <a href="https://en.wikipedia.org/wiki/Riesz%E2%80%93Markov%E2%80%93Kakutani_representation_theorem">Riesz representation theorem</a>, every such functional is an integration against some measure $\mu \in M(I_{n})$, so that $L(h) = \int_{I_{n}} h \, d\mu$ for all $h$.

Since $L$ vanishes on $S$, and each ridge function $x \mapsto \sigma(w^{T}x + b)$ belongs to $S$, we get

$$ \int_{I_{n}} \sigma(w^{T} x + b) \, d\mu(x) = 0 \qquad \text{for all } w, b. $$

But $\sigma$ is *discriminatory*, so this forces $\mu = 0$, and therefore $L = 0$, contradicting the fact that $L$ was chosen non-zero. Hence no proper closed subspace can contain $S$, which means $\overline{S} = C(I_{n})$. $\;\;\blacksquare$

The whole argument is a duality trick : *density* of a subspace is equivalent to the *absence* of a non-zero functional annihilating it, and the discriminatory property is precisely what rules out such an annihilator.

##1. Beyond sigmoids
Cybenko's proof is tailored to sigmoidal activations, but the phenomenon is far more general. Hornik {% cite Hornik1991 %} showed that the sigmoidal shape is not what matters : what matters is mere non-linearity and boundedness.

<div class="theorem" text="Hornik, 1991">
	Multilayer feedforward networks with a single hidden layer and any continuous, bounded and non-constant activation function are universal approximators on compact sets.
</div>

The final word belongs to Leshno *et al.* {% cite Leshno1993 %}, who pinned down the exact frontier. Their condition is beautifully simple and, retrospectively, explains why practitioners had so much freedom in their choice of activation.

<div class="theorem" text="Leshno et al., 1993">
	A feedforward network with a single hidden layer and a continuous activation function $\sigma$ is a universal approximator (dense in $C(I_{n})$) <b>if and only if</b> $\sigma$ is <b>not a polynomial</b>.
</div>

This is the reason the ubiquitous <a href="https://en.wikipedia.org/wiki/Rectifier_(neural_networks)">ReLU</a> $\sigma(t) = \max(0, t)$, which is neither bounded nor sigmoidal, is nonetheless a perfectly valid universal approximator : it simply is not a polynomial. A network built out of polynomial activations, on the other hand, can only ever produce polynomials of bounded degree, and thus can never approximate, say, $\sin$ uniformly. Leshno's condition turns a folklore intuition into a theorem.

<div class="example">
	It is worth pausing on the polynomial exclusion. If $\sigma$ is a polynomial of degree $d$, then every $\sigma(w^{T}x + b)$ is a polynomial of degree at most $d$ in $x$, and so is any finite linear combination $\eqref{eq:network}$. The class $S$ then lives inside a finite dimensional subspace of $C(I_{n})$, which cannot be dense. Non-polynomiality is exactly what breaks this ceiling.
</div>

##1. What the theorem does not say
The Universal Approximation Theorem is often invoked as a slogan (*"neural networks can learn anything"*), and this is where the misunderstandings begin. Three caveats deserve to be spelled out.

- **It is an existence result, not a construction.** The theorem guarantees that *some* network of the form $\eqref{eq:network}$ approximates $ƒ$, but it says nothing about how to *find* the weights. Nothing here mentions <a href="https://en.wikipedia.org/wiki/Gradient_descent">gradient descent</a>, backpropagation, or whether the optimisation landscape is friendly. Representability and *learnability* are different problems.

- **It says nothing about size.** The width $N$ needed to reach accuracy $\varepsilon$ can be astronomically large, and in the worst case it grows exponentially with the input dimension $n$, the so called <a href="https://en.wikipedia.org/wiki/Curse_of_dimensionality">curse of dimensionality</a>. A theorem that allows a billion neurons is a weak practical guarantee.

- **It says nothing about generalisation.** Approximating $ƒ$ well on the training domain is not the same as behaving well on unseen data. The theorem lives in the world of *approximation*, not *statistical estimation*.

This is precisely the tension that motivates <a href="https://en.wikipedia.org/wiki/Deep_learning">deep</a> networks. Depth does not extend *what* can be represented, that battle was already won by a single layer, but it can drastically reduce *how many* neurons are needed. Certain functions expressible by a deep network of modest size provably require an exponentially wider shallow network to match. Universality tells us the destination exists ; depth is about getting there efficiently.

##1. A historical cousin
It would be unfair to close without mentioning that the idea predates neural networks. In 1957, answering a version of Hilbert's thirteenth problem, Kolmogorov {% cite Kolmogorov1957 %} proved that *every* continuous function of several variables can be written as a superposition of continuous functions of a **single** variable and addition,

$$ ƒ(x_{1}, \dots, x_{n}) = \sum_{q=0}^{2n} \Phi_{q}\!\left( \sum_{p=1}^{n} \phi_{q, p}(x_{p}) \right). $$

The resemblance to a two layer network is uncanny : an inner layer of univariate maps $\phi_{q,p}$ feeding an outer layer of univariate maps $\Phi_{q}$. The catch is that the functions produced by Kolmogorov's theorem are highly irregular and depend on $ƒ$ in a non-constructive way, so it is an *exact representation* result rather than an *approximation* one. Still, it is a striking reminder that the expressive power of layered univariate non-linearities was understood, in a different language, decades before it was rediscovered by the connectionists. I will devote a separate post to the [Kolmogorov superposition theorem]({{ site.baseurl }}/blog/mathematics/kolmogorov-superposition), as it deserves a treatment of its own.

##1. Conclusion
The Universal Approximation Theorem is a reassuring foundation : it certifies that shallow feedforward networks are, in principle, as expressive as we could hope, capable of matching any continuous function on a compact set to arbitrary precision. Its proof is a clean piece of functional analysis resting on Hahn–Banach and Riesz duality, and its scope was later sharpened to the elegant *"anything but a polynomial"* criterion. But the theorem is a statement about *possibility*, not *practicality*. It says nothing about the number of neurons, the difficulty of training, or generalisation to new data, precisely the questions that occupy modern deep learning. Knowing that the target is reachable is comforting ; the rest of the field is about learning to reach it.


References
----------
{% bibliography --cited %}

[^1]: The theorem is usually stated on the unit cube $I_n = [0,1]^n$, but compactness is the only property that matters : the same result holds on any compact subset of $\Bbb{R}^n$.
