---
layout: post
title: Gradient Descent and Optimisation
date: 2020-09-10 12:00
comments: true
external-url:
categories: [Machine Learning]
tags: [machine_learning, optimisation, calculus]
permalink: /blog/machine-learning/gradient-descent
---


> Imagine you are lost on a foggy mountain and want to reach the valley. You cannot see far, but you can feel the slope under your feet. A sensible plan : take a small step in the steepest downhill direction, then feel around again, and repeat. Do this patiently and you will end up at the bottom. That simple idea, *follow the slope downhill*, is **gradient descent**, the optimisation workhorse behind almost all of machine learning. It is how a neural network turns "here is how wrong you are" into "here is how to be a little less wrong". This post builds it from the hiker's intuition up to the update rule $\theta \leftarrow \theta - \eta\, \nabla L(\theta)$, and explains the one knob, the learning rate, that makes or breaks it.

##1. The problem : minimise a loss
Training a model means choosing its parameters $\theta$ (all the weights) so that its predictions are good. We measure "badness" with a **loss function** $L(\theta)$, a single number that is large when the model is wrong and small when it is right. Learning is then just **minimisation** :

$$ \begin{equation} \theta^{\ast} = \arg\min_{\theta}\; L(\theta). \label{eq:argmin} \end{equation} $$

For anything but the simplest models we cannot solve this with a formula. There are millions of parameters and $L$ is a wildly complicated landscape. So instead of jumping to the answer, we **walk** to it.

##1. The gradient : which way is downhill ?
The tool that tells us the slope is the **gradient** $\nabla L(\theta)$, the vector of partial derivatives

$$ \begin{equation} \nabla L(\theta) = \left( \frac{\partial L}{\partial \theta_1}, \frac{\partial L}{\partial \theta_2}, \dots, \frac{\partial L}{\partial \theta_n} \right). \label{eq:grad} \end{equation} $$

<div class="definition">
	The <b>gradient</b> $\nabla L(\theta)$ points in the direction of <b>steepest ascent</b> of $L$, and its length is how steep that climb is. Therefore $-\nabla L(\theta)$ points in the direction of steepest <b>descent</b>.
</div>

*In plain words : the gradient is the compass that always points straight uphill. To go down, we simply walk the opposite way.* That is the whole trick.

##1. The update rule
Gradient descent turns "walk downhill" into one line of maths [^1]. Starting from a guess $\theta_0$, repeat :

$$ \begin{equation} \theta_{t+1} = \theta_{t} - \eta\, \nabla L(\theta_{t}). \label{eq:gd} \end{equation} $$

Each term has a plain meaning : $\nabla L$ is *which way is uphill*, the minus sign says *go the other way*, and $\eta > 0$ (the **learning rate**) is *how big a step to take*. The animation below drops a ball on a loss curve ; at every step it reads the local slope and moves against it, settling into the valley.

<figure><center>
<svg viewBox="0 0 640 300" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g1-curve{ fill:none; stroke:#4a6da7; stroke-width:2.6; }
    .g1-axis{ stroke:#bbb; stroke-width:1; }
    .g1-ball{ fill:#b5651d; }
    .g1-step{ fill:#b5651d; opacity:.35; }
    .g1-lab{ font-size:12px; fill:#7a7a7a; } .g1-lz{ font-size:13px; fill:#2b2b2b; }
  </style>
  <line class="g1-axis" x1="40" y1="250" x2="600" y2="250"/>
  <line class="g1-axis" x1="60" y1="30" x2="60" y2="255"/>
  <text class="g1-lab" x="30" y="140" text-anchor="middle" transform="rotate(-90 30 140)">loss L(θ)</text>
  <text class="g1-lab" x="330" y="278" text-anchor="middle">parameter θ  →</text>
  <!-- bowl-shaped loss curve: a parabola from (80,60) down to (330,235) up to (600,80) -->
  <path class="g1-curve" d="M80,60 Q330,360 600,80"/>
  <!-- static faded step markers along the descent (right side coming down) -->
  <circle class="g1-step" cx="560" cy="105" r="6"/>
  <circle class="g1-step" cx="500" cy="150" r="6"/>
  <circle class="g1-step" cx="450" cy="185" r="6"/>
  <circle class="g1-step" cx="410" cy="208" r="6"/>
  <circle class="g1-step" cx="380" cy="221" r="6"/>
  <circle class="g1-step" cx="358" cy="229" r="6"/>
  <!-- animated ball following the curve down to the minimum -->
  <circle class="g1-ball" r="8">
    <animateMotion dur="4s" repeatCount="indefinite"
      keyPoints="0;0.28;0.42;0.52;0.60;0.66;0.70;0.70;0"
      keyTimes="0;0.12;0.24;0.36;0.5;0.64;0.8;0.92;1"
      calcMode="linear"
      path="M80,60 Q330,360 600,80"/>
  </circle>
  <text class="g1-lz" x="330" y="250" text-anchor="middle">minimum</text>
</svg>
<figcaption><b>Figure 1 -</b> Gradient descent on a 1-D loss. The ball repeatedly moves against the slope $\eqref{eq:gd}$, taking large steps where the curve is steep and small steps as it flattens near the minimum.</figcaption>
</center></figure>

##1. The learning rate : the one knob that matters
The step size $\eta$ is deceptively important. Get it wrong and the whole thing fails in one of two opposite ways.

- **Too small.** Progress is safe but painfully slow ; you inch down the mountain and training takes forever.
- **Too large.** You leap past the valley to the far slope, then leap back even further, **overshooting** and possibly **diverging** to infinity.
- **Just right.** A handful of well-sized steps land you near the bottom.

<figure><center>
<svg viewBox="0 0 640 260" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g2-cv{ fill:none; stroke:#c9cdd6; stroke-width:2.2; }
    .g2-p1{ fill:none; stroke:#5aa06a; stroke-width:2; marker-end:url(#g2-g); }
    .g2-p2{ fill:none; stroke:#4a6da7; stroke-width:2; marker-end:url(#g2-b); }
    .g2-p3{ fill:none; stroke:#b5651d; stroke-width:2; marker-end:url(#g2-o); }
    .g2-tt{ font-size:12px; } .g2-cap{ font-size:11px; fill:#7a7a7a; text-anchor:middle; }
  </style>
  <defs>
    <marker id="g2-g" markerWidth="8" markerHeight="8" refX="5" refY="3" orient="auto"><path d="M0,0 L5,3 L0,6 Z" fill="#5aa06a"/></marker>
    <marker id="g2-b" markerWidth="8" markerHeight="8" refX="5" refY="3" orient="auto"><path d="M0,0 L5,3 L0,6 Z" fill="#4a6da7"/></marker>
    <marker id="g2-o" markerWidth="8" markerHeight="8" refX="5" refY="3" orient="auto"><path d="M0,0 L5,3 L0,6 Z" fill="#b5651d"/></marker>
  </defs>
  <!-- three small bowls -->
  <g>
    <path class="g2-cv" d="M20,40 Q100,230 180,40"/>
    <text class="g2-tt" x="100" y="24" text-anchor="middle" fill="#5aa06a">η too small</text>
    <!-- many tiny steps -->
    <path class="g2-p1" d="M170,55 L162,72 L155,88 L149,103 L144,116 L140,127 L137,136"/>
    <text class="g2-cap" x="100" y="250">slow crawl</text>
  </g>
  <g>
    <path class="g2-cv" d="M230,40 Q310,230 390,40"/>
    <text class="g2-tt" x="310" y="24" text-anchor="middle" fill="#4a6da7">η just right</text>
    <path class="g2-p2" d="M378,58 L330,150 L305,175 L312,168"/>
    <text class="g2-cap" x="310" y="250">few good steps</text>
  </g>
  <g>
    <path class="g2-cv" d="M440,40 Q520,230 600,40"/>
    <text class="g2-tt" x="520" y="24" text-anchor="middle" fill="#b5651d">η too large</text>
    <!-- overshoot zigzag growing -->
    <path class="g2-p3" d="M585,60 L505,150 L455,95 L560,175 L440,70"/>
    <text class="g2-cap" x="520" y="250">overshoots / diverges</text>
  </g>
</svg>
<figcaption><b>Figure 2 -</b> The effect of the learning rate $\eta$. Too small crawls, too large overshoots and can diverge, and a good value converges in a few steps. Tuning $\eta$ is the central practical skill of training.</figcaption>
</center></figure>

##1. Stochastic gradient descent
There is a catch hiding in $\eqref{eq:gd}$ : computing $\nabla L$ exactly means averaging the error over the **entire** dataset, which can be millions of examples, for *every single step*. That is far too slow.

The fix is beautifully lazy : estimate the gradient from a small random **mini-batch** of examples instead. This is **stochastic gradient descent** (SGD) {% cite Robbins1951 %}. The estimate is noisy, so the path down the mountain wobbles rather than flows, but each step is thousands of times cheaper, and the noise even helps to escape shallow traps.

<div class="example">
	With $1{,}000{,}000$ training examples, one exact gradient step reads all million. SGD with a mini-batch of $256$ takes a slightly wobbly step after reading just $256$, so in the time of one exact step it can take roughly $4{,}000$ stochastic ones. Noisier, but overwhelmingly faster in practice.
</div>

Modern optimisers refine this further. **Momentum** remembers past steps and keeps rolling, damping the wobble like a heavy ball. **Adam** {% cite Kingma2015 %} adapts a separate step size for each parameter from the recent history of gradients. They are all, at heart, still $\eqref{eq:gd}$ with a smarter step.

##1. The bumpy reality : non-convex landscapes
The bowl in Figure 1 is a **convex** loss : one valley, and downhill always leads to it. Neural-network losses are **non-convex**, a mountain range of many valleys, ridges and saddle points, so gradient descent only guarantees a *local* minimum, not the global best.

Remarkably, this matters far less than one would fear. In the huge parameter spaces of deep learning, most local minima turn out to be nearly as good as the global one, and the truly bad configurations are rare saddle points that noise helps to slip past. This empirical grace is a big part of *why* deep learning works, and it complements the [Universal Approximation Theorem]({{ site.baseurl }}/blog/mathematics/universal-approximation-theorem) : that theorem promises a good set of weights *exists*, and gradient descent is the procedure that actually goes and *finds* one.

##1. Conclusion
Gradient descent is the hiker's rule written in calculus : read the slope with the gradient, step against it, and control your stride with the learning rate. Everything else, mini-batches, momentum, Adam, non-convex landscapes, is a refinement of that one move, $\theta \leftarrow \theta - \eta \nabla L$. But there is a missing piece : for a deep network, how do we even *compute* the gradient $\nabla L$ over millions of tangled weights ? That is the job of a wonderfully efficient algorithm called [backpropagation]({{ site.baseurl }}/blog/machine-learning/backpropagation), the subject of the next post.


References
----------
{% bibliography --cited %}

[^1]: Gradient descent dates back to Cauchy in 1847. What is new in the deep-learning era is not the method but the scale, billions of parameters, and the automatic computation of the gradient by backpropagation.
