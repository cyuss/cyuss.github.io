---
layout: post
title: Backpropagation, How Neural Nets Actually Learn
date: 2022-04-20 12:00
comments: true
external-url:
categories: [Machine Learning]
tags: [machine_learning, deep_learning, calculus]
permalink: /blog/machine-learning/backpropagation
---


> Everyone is told that neural networks learn by "backpropagation", and everyone is then shown a wall of matrix equations bristling with transposes and tiny delta symbols. Let us throw that away and do something better : take one tiny network, put **actual numbers** through it, and watch the gradients being computed by hand. You will see that backpropagation is not an algorithm you need to memorise, it is just the **chain rule** walking backwards through a diagram, where each little box does one embarrassingly simple local calculation and whispers the result to its neighbour. By the end you will be able to backpropagate on paper.

##1. A network is a graph of tiny operations
Forget "layers" for a moment. Any computation, including a neural network, can be drawn as a **computational graph** : a diagram where each node is one small operation (a multiply, an add, a squashing function) and arrows carry numbers from one node to the next.

Take the smallest possible "network" : a single neuron that multiplies its input $x$ by a weight $w$, adds a bias $b$, squashes the result with the sigmoid $\sigma(z) = \frac{1}{1+e^{-z}}$, and is scored against a target $y$ by the squared-error loss. As a graph, that is just five little boxes :

$$ \begin{equation} x \xrightarrow{\;\times w\;} \; \xrightarrow{\;+\,b\;} z \xrightarrow{\;\sigma\;} a \xrightarrow{\;(a-y)^2\;} L. \label{eq:graph} \end{equation} $$

That is the whole model. Learning means answering one question : if I nudge $w$ a little, how does the loss $L$ change ? That number, $\frac{\partial L}{\partial w}$, is the gradient, and [gradient descent]({{ site.baseurl }}/blog/machine-learning/gradient-descent) will use it to improve $w$.

##1. The forward pass : just compute
First we run the graph left to right with concrete numbers. Let us pick $x = 1$, $w = 0.5$, $b = 0$, and target $y = 1$. Nothing clever here, we simply evaluate each box and remember its output (we will need those outputs in a minute).

<figure><center>
<svg viewBox="0 0 640 220" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .ed{ stroke:#4a6da7; stroke-width:2; fill:none; opacity:.55; }
    .op{ fill:#f6f7fb; stroke:#4a6da7; stroke-width:1.6; }
    .inp{ fill:#f3faf4; stroke:#5aa06a; stroke-width:1.6; }
    .lo{ fill:#faf6f2; stroke:#b5651d; stroke-width:1.6; }
    .nm{ font-size:15px; fill:#2b2b2b; text-anchor:middle; } .it{ font-style:italic; }
    .vl{ font-size:12px; fill:#4a6da7; text-anchor:middle; }
    .fdot{ fill:#4a6da7; }
  </style>
  <!-- edges -->
  <path class="ed" d="M92,60 L204,95"/>
  <path class="ed" d="M92,125 L204,105"/>
  <path class="ed" d="M256,100 L336,110"/>
  <path class="ed" d="M92,190 L336,128"/>
  <path class="ed" d="M392,118 L452,118"/>
  <path class="ed" d="M508,118 L556,118"/>
  <!-- input nodes -->
  <circle class="inp" cx="70" cy="60"  r="20"/><text class="nm it" x="70" y="65">x</text><text class="vl" x="70" y="30">1</text>
  <circle class="inp" cx="70" cy="125" r="20"/><text class="nm it" x="70" y="130">w</text><text class="vl" x="30" y="130">0.5</text>
  <circle class="inp" cx="70" cy="190" r="20"/><text class="nm it" x="70" y="195">b</text><text class="vl" x="30" y="195">0</text>
  <!-- mul -->
  <circle class="op" cx="230" cy="100" r="22"/><text class="nm" x="230" y="106">×</text><text class="vl" x="230" y="68">0.5</text>
  <!-- add -->
  <circle class="op" cx="364" cy="118" r="22"/><text class="nm" x="364" y="124">+</text><text class="vl" x="364" y="86">z = 0.5</text>
  <!-- sigma -->
  <circle class="op" cx="480" cy="118" r="22"/><text class="nm" x="480" y="124">σ</text><text class="vl" x="480" y="86">a = 0.622</text>
  <!-- loss -->
  <circle class="lo" cx="580" cy="118" r="24"/><text class="nm" x="580" y="123">L</text><text class="vl" x="580" y="84">0.143</text>
  <!-- moving forward dot -->
  <circle class="fdot" r="6"><animateMotion dur="3s" repeatCount="indefinite" path="M70,60 L230,100 L364,118 L480,118 L580,118"/></circle>
  <text class="vl" x="320" y="205" fill="#7a7a7a" font-size="12">forward pass : evaluate each box left → right (y = 1 enters the loss)</text>
</svg>
<figcaption><b>Figure 1 -</b> The forward pass. Each node computes its output from its inputs : $wx = 0.5$, then $+b = 0.5 = z$, then $\sigma(z) = 0.622 = a$, then $(a-y)^2 = 0.143 = L$. We store every intermediate value.</figcaption>
</center></figure>

##1. The one rule of backpropagation
Now the only idea you need. The chain rule says that to get the gradient of $L$ with respect to something early in the graph, you **multiply the local derivatives along the path** connecting them. Backpropagation organises this multiplication cleverly : it flows a single number, *"how much does $L$ change per unit change of my output ?"*, backwards through the graph. Call the number arriving at a node its **upstream gradient**. Then every node obeys one rule :

<div class="theorem" text="the backprop rule">
	<b>upstream gradient into an input</b> $=$ <b>upstream gradient at the output</b> $\times$ <b>the node's own local derivative</b>.
	$$ \frac{\partial L}{\partial(\text{input})} = \frac{\partial L}{\partial(\text{output})}\cdot\frac{\partial(\text{output})}{\partial(\text{input})}. $$
</div>

*In plain words : each box is a little worker who knows only one thing, how sensitive its own output is to its own input. It takes the "blame" handed down from the right, scales it by that local sensitivity, and passes it further left. No box ever needs to understand the whole network.* Here are the local derivatives our five boxes need, each a one-liner from calculus :

$$ \begin{array}{ l c l }
\text{loss } (a-y)^2 & : & \dfrac{\partial L}{\partial a} = 2(a - y) \\[4pt]
\text{sigmoid } a=\sigma(z) & : & \dfrac{\partial a}{\partial z} = a(1-a) \\[4pt]
\text{add } z = (wx) + b & : & \dfrac{\partial z}{\partial b} = 1,\quad \dfrac{\partial z}{\partial (wx)} = 1 \\[4pt]
\text{multiply } wx & : & \dfrac{\partial (wx)}{\partial w} = x,\quad \dfrac{\partial (wx)}{\partial x} = w
\end{array} $$

##1. The backward pass : the same graph, in reverse
Start at the right with the trivial fact $\frac{\partial L}{\partial L} = 1$, and apply the rule box by box, reusing the values we stored during the forward pass.

$$ \begin{aligned}
\frac{\partial L}{\partial a} &= 2(a-y) = 2(0.622 - 1) = -0.755, \\
\frac{\partial L}{\partial z} &= \frac{\partial L}{\partial a}\cdot a(1-a) = -0.755 \times (0.622)(0.378) = -0.177, \\
\frac{\partial L}{\partial b} &= \frac{\partial L}{\partial z}\cdot 1 = -0.177, \\
\frac{\partial L}{\partial w} &= \frac{\partial L}{\partial z}\cdot x = -0.177 \times 1 = -0.177, \\
\frac{\partial L}{\partial x} &= \frac{\partial L}{\partial z}\cdot w = -0.177 \times 0.5 = -0.089.
\end{aligned}$$

<figure><center>
<svg viewBox="0 0 640 220" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .edb{ stroke:#b5651d; stroke-width:2; fill:none; opacity:.5; }
    .op2{ fill:#f6f7fb; stroke:#4a6da7; stroke-width:1.6; }
    .inp2{ fill:#f3faf4; stroke:#5aa06a; stroke-width:1.6; }
    .lo2{ fill:#faf6f2; stroke:#b5651d; stroke-width:1.6; }
    .nm2{ font-size:15px; fill:#2b2b2b; text-anchor:middle; } .it{ font-style:italic; }
    .gr{ font-size:12px; fill:#b5651d; text-anchor:middle; font-weight:bold; }
    .bdot{ fill:#b5651d; }
  </style>
  <path class="edb" d="M92,60 L204,95"/>
  <path class="edb" d="M92,125 L204,105"/>
  <path class="edb" d="M256,100 L336,110"/>
  <path class="edb" d="M92,190 L336,128"/>
  <path class="edb" d="M392,118 L452,118"/>
  <path class="edb" d="M508,118 L556,118"/>
  <circle class="inp2" cx="70" cy="60"  r="20"/><text class="nm2 it" x="70" y="65">x</text><text class="gr" x="70" y="30">−0.089</text>
  <circle class="inp2" cx="70" cy="125" r="20"/><text class="nm2 it" x="70" y="130">w</text><text class="gr" x="24" y="130">−0.177</text>
  <circle class="inp2" cx="70" cy="190" r="20"/><text class="nm2 it" x="70" y="195">b</text><text class="gr" x="24" y="195">−0.177</text>
  <circle class="op2" cx="230" cy="100" r="22"/><text class="nm2" x="230" y="106">×</text>
  <circle class="op2" cx="364" cy="118" r="22"/><text class="nm2" x="364" y="124">+</text>
  <circle class="op2" cx="480" cy="118" r="22"/><text class="nm2" x="480" y="124">σ</text><text class="gr" x="440" y="86">−0.177</text>
  <circle class="lo2" cx="580" cy="118" r="24"/><text class="nm2" x="580" y="123">L</text><text class="gr" x="580" y="84">−0.755</text>
  <!-- moving backward dot -->
  <circle class="bdot" r="6"><animateMotion dur="3s" repeatCount="indefinite" path="M580,118 L480,118 L364,118 L230,100 L70,125"/></circle>
  <text class="gr" x="320" y="205" fill="#7a7a7a" font-size="12" font-weight="normal">backward pass : the gradient flows right → left, scaled at each box</text>
</svg>
<figcaption><b>Figure 2 -</b> The backward pass over the <i>same</i> graph. The orange number reaching each node is $\partial L / \partial(\text{that value})$. Each box multiplied the gradient handed to it by its own local derivative, precisely the backprop rule.</figcaption>
</center></figure>

And that is backpropagation, complete, on a real example. We now know that increasing $w$ by a tiny amount *decreases* the loss (the gradient $-0.177$ is negative), so gradient descent will nudge $w$ **up** : $w \leftarrow w - \eta(-0.177)$. Do this repeatedly and the neuron learns.

<div class="example">
	Notice the <b>fork</b> at the multiply node : it sent gradient to <i>both</i> $w$ and $x$, each scaled by a different local derivative ($x$ and $w$ respectively). This is the general rule for a node with several inputs, give each input the upstream gradient times its own local derivative. When a value is used in several places, its gradients from those places simply <b>add up</b>.
</div>

##1. Why do it backwards, and why it is fast
Why flow right-to-left instead of left-to-right ? Because there is one loss at the far right but many parameters at the left. Sweeping backwards from that single output computes the gradient for **every** parameter in one pass, sharing all the intermediate work. Computing it forwards would mean redoing the whole chain separately for each weight.

This sharing is the same trick as [memoization]({{ site.baseurl }}/blog/python/memoization) : the upstream gradient at a node is computed **once** and reused for all the inputs feeding it. That is why the backward pass costs about the same as the forward pass, even for a network with billions of weights. Without it, training modern models would be flatly impossible. Under its formal name, *reverse-mode automatic differentiation*, this is exactly what PyTorch and TensorFlow do for you when you call `.backward()`. [^1]

##1. One honest warning : vanishing gradients
Look again at the backward pass : the gradient is a **product** of local derivatives, one per box on the path. The sigmoid's local derivative $a(1-a)$ is at most $0.25$. Chain a dozen sigmoids and the gradient reaching the earliest layers is multiplied by $0.25$ a dozen times, roughly $0.25^{12} \approx 6\times10^{-8}$, effectively zero. Those early layers barely learn : the notorious **vanishing gradient** problem.

This single observation, that a long product of small numbers collapses, explains a surprising amount of modern design : the pairing of [softmax with cross-entropy]({{ site.baseurl }}/blog/machine-learning/softmax-cross-entropy) at the output layer (whose gradient $p-y$ never shrinks, unlike the squared error's), the ReLU activation (local derivative exactly $1$ on the positive side, so the product does not shrink), careful weight initialisation, and the *residual connections* of the [Transformer]({{ site.baseurl }}/blog/machine-learning/transformers), which hand the gradient a clean shortcut straight back to the early layers.

##1. Conclusion
Backpropagation deserves to feel easy, because it is : draw the computation as a graph, run it forwards once and remember the intermediate values, then sweep backwards multiplying by one local derivative at each box. The chain rule does all the work ; backprop is merely the bookkeeping that reuses shared results so the whole gradient falls out in a single pass. Feed those gradients to [gradient descent]({{ site.baseurl }}/blog/machine-learning/gradient-descent) and repeat a few million times, and the [Universal Approximation Theorem]({{ site.baseurl }}/blog/mathematics/universal-approximation-theorem)'s promise, that a good network *exists*, is turned into a network you actually *have*.


References
----------
{% bibliography --cited %}

[^1]: Popularised for neural networks by Rumelhart, Hinton and Williams {% cite Rumelhart1986 %} in 1986, though the reverse-mode differentiation at its core was known earlier, notably in Paul Werbos's 1974 thesis. Every modern framework builds it in as automatic differentiation.
