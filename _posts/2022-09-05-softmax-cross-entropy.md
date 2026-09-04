---
layout: post
title: Softmax and Cross-Entropy, Explained
date: 2022-09-05 12:00
comments: true
external-url:
categories: [Machine Learning]
tags: [machine_learning, deep_learning, probability, information_theory]
permalink: /blog/machine-learning/softmax-cross-entropy
---


> A neural network does not answer questions, it produces **numbers**. Ask it to recognise an animal and the last layer hands you something like $(2.0,\; 1.0,\; 0.1)$ for *cat, dog, bird*. That is not an answer, it is a mood. Two small functions turn that mood into a decision and, crucially, into a *lesson* : **softmax** converts the raw scores into honest probabilities, and **cross-entropy** measures how wrong those probabilities were. Together they are the last two boxes of almost every classifier ever trained, and they hide a small miracle : their combined gradient is simply *predicted minus true*. This post builds both from scratch, with numbers, and shows exactly where that miracle comes from.

##1. The problem : scores are not probabilities
The final layer of a classifier outputs one number per class. These raw numbers are called **logits**. In our example the network is leaning towards *cat* :

$$ \begin{equation} z = (z_{\text{cat}},\, z_{\text{dog}},\, z_{\text{bird}}) = (2.0,\; 1.0,\; 0.1). \label{eq:logits} \end{equation} $$

We would like to say *"cat with 66 % confidence"*, but $\eqref{eq:logits}$ cannot be read as probabilities : the numbers do not sum to $1$, and nothing stops a logit from being negative. We need a converter that takes any list of real numbers and returns a proper probability distribution : all entries positive, summing to one, and preserving the ranking (whoever scored highest must stay the most likely).

The naive fix, dividing each score by the total, fails immediately on negative numbers : with $z = (-1, 2)$ the sum is $1$ and we would report a probability of $-100\,\%$ for the first class. We need something that makes every number positive **before** normalising.

##1. Softmax : exponentiate, then normalise
The exponential is exactly the tool for that. It is always positive, and it is increasing, so it never reorders anything. Apply it to each logit and divide by the total :

<div class="definition">
	The <b>softmax</b> function turns a vector of logits $z = (z_1,\dots,z_K)$ into a probability distribution :
	$$ \operatorname{softmax}(z)_i \;=\; p_i \;=\; \frac{e^{z_i}}{\sum_{k=1}^{K} e^{z_k}}, \qquad p_i > 0, \quad \sum_{i=1}^{K} p_i = 1. $$
</div>

*In plain words : give every class a number of lottery tickets equal to $e^{z}$, then a class's probability is its share of all the tickets in the urn. Because the ticket count grows exponentially with the score, a slightly better score buys a lot more tickets.* On our example :

$$ \begin{aligned}
e^{2.0} &= 7.389, & e^{1.0} &= 2.718, & e^{0.1} &= 1.105, & \text{sum} &= 11.212, \\
p_{\text{cat}} &= \tfrac{7.389}{11.212} = 0.659, & p_{\text{dog}} &= \tfrac{2.718}{11.212} = 0.242, & p_{\text{bird}} &= \tfrac{1.105}{11.212} = 0.099. &&
\end{aligned} $$

<figure><center>
<svg viewBox="0 0 640 240" width="100%" style="max-width:620px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g1-lb{ fill:#8fa5c8; stroke:#4a6da7; stroke-width:1.2; }
    .g1-pb{ fill:#e0a878; stroke:#b5651d; stroke-width:1.2; }
    .g1-ax{ stroke:#bdbdbd; stroke-width:1.2; }
    .g1-cl{ font-size:12px; fill:#5a5a5a; text-anchor:middle; }
    .g1-vl{ font-size:12px; fill:#2b2b2b; text-anchor:middle; }
    .g1-hd{ font-size:13px; fill:#4a6da7; text-anchor:middle; font-style:italic; }
    .g1-hd2{ font-size:13px; fill:#b5651d; text-anchor:middle; font-style:italic; }
    .g1-arw{ stroke:#7a7a7a; stroke-width:1.6; fill:none; }
    .g1-op{ font-size:13px; fill:#5a5a5a; text-anchor:middle; }
    .g1-fl{ fill:#5aa06a; }
  </style>
  <!-- left panel : logits -->
  <text class="g1-hd" x="115" y="26">logits z (raw scores)</text>
  <line class="g1-ax" x1="35" y1="190" x2="200" y2="190"/>
  <rect class="g1-lb" x="52"  y="130" width="34" height="60"/><text class="g1-vl" x="69"  y="122">2.0</text><text class="g1-cl" x="69"  y="206">cat</text>
  <rect class="g1-lb" x="100" y="160" width="34" height="30"/><text class="g1-vl" x="117" y="152">1.0</text><text class="g1-cl" x="117" y="206">dog</text>
  <rect class="g1-lb" x="148" y="187" width="34" height="3"/> <text class="g1-vl" x="165" y="179">0.1</text><text class="g1-cl" x="165" y="206">bird</text>
  <!-- middle : the two operations -->
  <path class="g1-arw" d="M215,150 L300,150"/><path class="g1-arw" d="M300,150 l-8,-5 M300,150 l-8,5"/>
  <text class="g1-op" x="257" y="140">e ᶻ</text>
  <text class="g1-op" x="257" y="172" font-size="11">7.389 · 2.718 · 1.105</text>
  <path class="g1-arw" d="M320,150 L405,150"/><path class="g1-arw" d="M405,150 l-8,-5 M405,150 l-8,5"/>
  <text class="g1-op" x="362" y="140">÷ 11.212</text>
  <text class="g1-op" x="362" y="172" font-size="11">(the total)</text>
  <!-- travelling dot -->
  <circle class="g1-fl" r="5"><animateMotion dur="3.4s" repeatCount="indefinite" path="M215,150 L405,150"/></circle>
  <!-- right panel : probabilities -->
  <text class="g1-hd2" x="520" y="26">probabilities p (sum = 1)</text>
  <line class="g1-ax" x1="425" y1="190" x2="615" y2="190"/>
  <rect class="g1-pb" x="446" y="104" width="34" height="86"/><text class="g1-vl" x="463" y="96"  fill="#b5651d">0.659</text><text class="g1-cl" x="463" y="206">cat</text>
  <rect class="g1-pb" x="500" y="158" width="34" height="32"/><text class="g1-vl" x="517" y="150" fill="#b5651d">0.242</text><text class="g1-cl" x="517" y="206">dog</text>
  <rect class="g1-pb" x="554" y="177" width="34" height="13"/><text class="g1-vl" x="571" y="169" fill="#b5651d">0.099</text><text class="g1-cl" x="571" y="206">bird</text>
  <text class="g1-cl" x="320" y="230" fill="#7a7a7a">softmax : exponentiate to make everything positive, then normalise to make it a distribution</text>
</svg>
<figcaption><b>Figure 1 -</b> Softmax in two moves. The exponential turns arbitrary scores into positive "ticket counts", the division by their sum turns those counts into shares. The order of the classes is untouched.</figcaption>
</center></figure>

Two properties are worth keeping in your pocket. First, softmax depends **only on the differences** between logits : adding the same constant $c$ to every score changes nothing, since $e^{z_i+c} = e^{c}e^{z_i}$ and the $e^{c}$ cancels between numerator and denominator. Second, with only two classes it collapses to the familiar sigmoid of [backpropagation]({{ site.baseurl }}/blog/machine-learning/backpropagation) :

$$ \begin{equation} p_1 = \frac{e^{z_1}}{e^{z_1}+e^{z_2}} = \frac{1}{1 + e^{-(z_1 - z_2)}} = \sigma(z_1 - z_2). \label{eq:sigmoid} \end{equation} $$

So the sigmoid was never a different animal, it is softmax with $K=2$ looking at a single score gap.

<div class="example">
	<b>The name is a warning.</b> Softmax is <i>not</i> a smooth version of the <code>max</code> function, it is a smooth version of <code>argmax</code> : a hard argmax would return $(1,0,0)$, softmax returns the blurred $(0.659, 0.242, 0.099)$. That blur is the whole point, because a hard argmax has a derivative of zero everywhere and would tell <a href="{{ site.baseurl }}/blog/machine-learning/gradient-descent">gradient descent</a> absolutely nothing.
</div>

##1. Measuring the mistake : surprise, then cross-entropy
The network has now given an opinion. Suppose the picture was actually a **dog**. How bad is that opinion ? We need a loss, and it must respect one intuition : being confidently wrong should hurt far more than being hesitantly wrong.

Information theory gives the natural measure. Shannon {% cite Shannon1948 %} defines the **surprise** of an event you assigned probability $p$ as $-\log p$. Predict the truth with $p=1$ and you are not surprised at all ($-\log 1 = 0$) ; predict it with $p = 0.01$ and you are very surprised ($-\log 0.01 = 4.6$) ; predict it with $p \to 0$ and your surprise goes to infinity.

<figure><center>
<svg viewBox="0 0 640 250" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g2-axis{ stroke:#9a9a9a; stroke-width:1.3; }
    .g2-grid{ stroke:#e6e6e6; stroke-width:1; }
    .g2-crv{ stroke:#b5651d; stroke-width:2.4; fill:none; }
    .g2-tk{ font-size:11px; fill:#7a7a7a; text-anchor:middle; }
    .g2-tky{ font-size:11px; fill:#7a7a7a; text-anchor:end; }
    .g2-lbl{ font-size:12px; fill:#5a5a5a; text-anchor:middle; font-style:italic; }
    .g2-mk{ fill:#4a6da7; } .g2-mklb{ font-size:11px; fill:#4a6da7; text-anchor:middle; }
    .g2-dsh{ stroke:#4a6da7; stroke-width:1; stroke-dasharray:3 3; }
  </style>
  <line class="g2-grid" x1="70" y1="164" x2="600" y2="164"/>
  <line class="g2-grid" x1="70" y1="128" x2="600" y2="128"/>
  <line class="g2-grid" x1="70" y1="92"  x2="600" y2="92"/>
  <line class="g2-grid" x1="70" y1="56"  x2="600" y2="56"/>
  <line class="g2-axis" x1="70" y1="200" x2="610" y2="200"/>
  <line class="g2-axis" x1="70" y1="200" x2="70"  y2="26"/>
  <text class="g2-tky" x="63" y="204">0</text><text class="g2-tky" x="63" y="168">1</text><text class="g2-tky" x="63" y="132">2</text><text class="g2-tky" x="63" y="96">3</text><text class="g2-tky" x="63" y="60">4</text>
  <text class="g2-tk" x="70" y="218">0</text><text class="g2-tk" x="330" y="218">0.5</text><text class="g2-tk" x="590" y="218">1</text>
  <text class="g2-lbl" x="340" y="238">probability the model gave to the correct class</text>
  <text class="g2-lbl" x="18" y="112" transform="rotate(-90 18,112)">loss  −ln p</text>
  <path class="g2-crv" d="M75,34 L80,59 L96,92 L122,117 L174,142 L226,157 L278,167 L330,175 L382,182 L434,187 L486,192 L538,196 L590,200"/>
  <!-- marker at p = 0.242 -->
  <line class="g2-dsh" x1="196" y1="200" x2="196" y2="149"/>
  <line class="g2-dsh" x1="70"  y1="149" x2="196" y2="149"/>
  <circle class="g2-mk" cx="196" cy="149" r="4.5"/>
  <text class="g2-mklb" x="238" y="140">our dog : 1.417</text>
  <text class="g2-tk" x="196" y="218" fill="#4a6da7">0.242</text>
</svg>
<figcaption><b>Figure 2 -</b> The loss $-\ln p$ as a function of the probability given to the <i>correct</i> class. It is gentle when you were nearly right and explodes as your confidence in the truth approaches zero, which is exactly the asymmetry we wanted.</figcaption>
</center></figure>

Now write the truth as a **one-hot** vector : $y = (0, 1, 0)$ for *dog*, meaning "the true distribution puts all its mass on dog". The **cross-entropy** between the truth $y$ and the prediction $p$ averages the surprise over the true distribution :

<div class="definition">
	The <b>cross-entropy loss</b> between a target distribution $y$ and a predicted distribution $p$ over $K$ classes is
	$$ L(y, p) \;=\; -\sum_{i=1}^{K} y_i \log p_i . $$
	When $y$ is one-hot on the true class $c$, every term but one vanishes and this collapses to $L = -\log p_c$.
</div>

*In plain words : the loss only ever looks at the probability you assigned to the right answer, and asks how surprised you were to learn it was right.* For our misclassified dog :

$$ \begin{equation} L = -\log p_{\text{dog}} = -\log(0.242) = 1.417. \label{eq:loss} \end{equation} $$

Had the network said $p_{\text{dog}} = 0.9$ the loss would have been $0.105$ ; had it said $0.01$, it would have been $4.6$. That steepness is what makes the network flee confident mistakes.

<div class="example">
	<b>Why "cross"-entropy ?</b> The entropy $H(y) = -\sum_i y_i \log y_i$ is the surprise you would feel using the <i>true</i> distribution ; the cross-entropy is the surprise you feel using <i>your</i> distribution while the world follows $y$. Their difference is the Kullback–Leibler divergence {% cite Kullback1951 %} : $\;H(y,p) = H(y) + \mathrm{KL}(y \,\|\, p)$. For a one-hot target $H(y) = 0$, so minimising cross-entropy is <i>exactly</i> minimising the KL divergence between the truth and the prediction. The loss is not an arbitrary choice, it is the information-theoretic distance to the right answer.
</div>

##1. The miracle : the gradient is just $p - y$
So far, two reasonable functions. Here is why they are always used *together*. Chain them, $z \to p = \operatorname{softmax}(z) \to L = -\sum_i y_i\log p_i$, and ask what [backpropagation]({{ site.baseurl }}/blog/machine-learning/backpropagation) needs : the gradient of the loss with respect to the logits.

<div class="theorem" text="softmax + cross-entropy gradient">
	If $p = \operatorname{softmax}(z)$ and $L = -\sum_i y_i \log p_i$ with $\sum_i y_i = 1$, then
	$$ \frac{\partial L}{\partial z_k} = p_k - y_k . $$
</div>

No exponentials, no fractions, no special cases : **predicted minus true**. The derivation is two lines. Write $S = \sum_k e^{z_k}$, so $\log p_i = z_i - \log S$ and therefore

$$ L = -\sum_i y_i\,(z_i - \log S) = -\sum_i y_i z_i + \log S . $$

Differentiating with respect to one logit $z_k$, the first sum contributes $-y_k$, and the second gives $\frac{\partial \log S}{\partial z_k} = \frac{e^{z_k}}{S} = p_k$. Adding them :

$$ \begin{equation} \frac{\partial L}{\partial z_k} = p_k - y_k . \label{eq:grad} \end{equation} $$

On our example the gradient is $(0.659,\; 0.242 - 1,\; 0.099) = (0.659,\; -0.758,\; 0.099)$. Read it as an instruction : *lower the cat score, raise the dog score, lower the bird score a little* — each by an amount proportional to how badly it was misjudged. Gradient descent then does the obvious thing, $z \leftarrow z - \eta(p-y)$, and the network's next opinion is a little less wrong.

This is not a coincidence. The logarithm in the loss is precisely the inverse of the exponential in the softmax, and the two annihilate. Pair softmax with the squared-error loss instead and the gradient picks up a factor $p_k(1-p_k)$, which is nearly zero whenever the network is confident — including when it is *confidently wrong*, exactly the case you most need to fix. That is the vanishing-gradient trap of [backpropagation]({{ site.baseurl }}/blog/machine-learning/backpropagation) reappearing at the very last layer, and cross-entropy is what removes it.

##1. Temperature : the same scores, more or less bold
Because softmax reads only the *differences* between logits, scaling them all by a constant changes the shape of the distribution without changing the ranking. Divide the logits by a **temperature** $T$ before the softmax :

$$ \begin{equation} p_i(T) = \frac{e^{z_i/T}}{\sum_k e^{z_k/T}}. \label{eq:temp} \end{equation} $$

A small $T$ magnifies the gaps and makes the model decisive ; a large $T$ shrinks them towards a uniform distribution and makes it hesitant. In the limit, $T \to 0$ gives a hard argmax and $T \to \infty$ gives pure uniform noise.

<figure><center>
<svg viewBox="0 0 640 240" width="100%" style="max-width:560px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g3-tb{ fill:#e0a878; stroke:#b5651d; stroke-width:1.2; }
    .g3-ax2{ stroke:#bdbdbd; stroke-width:1.2; }
    .g3-cl2{ font-size:12px; fill:#5a5a5a; text-anchor:middle; }
    .g3-tt{ font-size:16px; fill:#4a6da7; text-anchor:middle; font-style:italic; }
    .g3-cp{ font-size:12px; fill:#7a7a7a; text-anchor:middle; }
  </style>
  <line class="g3-ax2" x1="180" y1="180" x2="460" y2="180"/>
  <rect class="g3-tb" x="205" y="94" width="46" height="86">
    <animate attributeName="height" dur="6s" repeatCount="indefinite" keyTimes="0;0.25;0.5;0.75;1" values="112;86;65;86;112"/>
    <animate attributeName="y"      dur="6s" repeatCount="indefinite" keyTimes="0;0.25;0.5;0.75;1" values="68;94;115;94;68"/>
  </rect>
  <rect class="g3-tb" x="297" y="148" width="46" height="32">
    <animate attributeName="height" dur="6s" repeatCount="indefinite" keyTimes="0;0.25;0.5;0.75;1" values="15;31.5;39.5;31.5;15"/>
    <animate attributeName="y"      dur="6s" repeatCount="indefinite" keyTimes="0;0.25;0.5;0.75;1" values="165;148.5;140.5;148.5;165"/>
  </rect>
  <rect class="g3-tb" x="389" y="167" width="46" height="13">
    <animate attributeName="height" dur="6s" repeatCount="indefinite" keyTimes="0;0.25;0.5;0.75;1" values="2.5;12.8;25.2;12.8;2.5"/>
    <animate attributeName="y"      dur="6s" repeatCount="indefinite" keyTimes="0;0.25;0.5;0.75;1" values="177.5;167.2;154.8;167.2;177.5"/>
  </rect>
  <text class="g3-cl2" x="228" y="196">cat</text><text class="g3-cl2" x="320" y="196">dog</text><text class="g3-cl2" x="412" y="196">bird</text>
  <text class="g3-tt" x="320" y="42">T = 0.5  (decisive)<animate attributeName="opacity" dur="6s" repeatCount="indefinite" keyTimes="0;0.2;0.25;0.75;0.8;1" values="1;1;0;0;1;1"/></text>
  <text class="g3-tt" x="320" y="42">T = 1  (as trained)<animate attributeName="opacity" dur="6s" repeatCount="indefinite" keyTimes="0;0.2;0.3;0.45;0.55;0.7;0.8;1" values="0;0;1;1;0;0;1;0"/></text>
  <text class="g3-tt" x="320" y="42">T = 2  (hesitant)<animate attributeName="opacity" dur="6s" repeatCount="indefinite" keyTimes="0;0.45;0.5;0.55;1" values="0;0;1;0;0"/></text>
  <text class="g3-cp" x="320" y="224">same logits (2.0, 1.0, 0.1), different temperature</text>
</svg>
<figcaption><b>Figure 3 -</b> Temperature reshapes the same three scores. Cooling sharpens the distribution towards the winner, heating flattens it towards uniform. Nothing about the model changed, only how boldly we read it.</figcaption>
</center></figure>

This is the `temperature` knob you set when sampling text from a [Transformer]({{ site.baseurl }}/blog/machine-learning/transformers) : low temperature gives safe, repetitive prose, high temperature gives creative and occasionally deranged prose. The same rescaling appears inside [attention]({{ site.baseurl }}/blog/machine-learning/attention-mechanism) as the division by $\sqrt{d_k}$, whose job is to keep the softmax away from the saturated regime where all the gradients die.

##1. Making it survive a computer
There is one practical trap. Logits of a few hundred are common in a large model, and $e^{800}$ overflows to infinity in floating point, turning the whole distribution into `NaN`. The fix uses the shift invariance we noticed earlier : subtracting a constant from every logit changes nothing mathematically, so subtract the largest one.

$$ \begin{equation} p_i = \frac{e^{z_i - m}}{\sum_k e^{z_k - m}}, \qquad m = \max_k z_k . \label{eq:stable} \end{equation} $$

Now the biggest exponent is $e^{0} = 1$ and every other is smaller, so nothing can overflow. This is the **log-sum-exp trick**, and it is why the loss is computed in log space directly :

$$ L = -\log p_c = -(z_c - m) + \log\!\sum_k e^{z_k - m}. $$

It is also why every framework offers a *fused* operation that takes the raw logits, never the probabilities : `torch.nn.CrossEntropyLoss` and `tf.nn.softmax_cross_entropy_with_logits` both apply the softmax internally. Computing `softmax` yourself and then feeding it to a separate `log` is the single most common beginner bug in this corner of deep learning : it throws away the numerical stability *and* the clean $p-y$ gradient of $\eqref{eq:grad}$.

```python
import numpy as np

def softmax(z):
    z = z - z.max()            # subtract max : no overflow
    e = np.exp(z)
    return e / e.sum()

def cross_entropy(z, c):       # c = index of the true class
    m = z.max()
    return -(z[c] - m) + np.log(np.exp(z - m).sum())

z = np.array([2.0, 1.0, 0.1])
p = softmax(z)                 # [0.659, 0.242, 0.099]
loss = cross_entropy(z, 1)     # 1.417  (truth = dog)
grad = p - np.array([0., 1., 0.])   # [0.659, -0.758, 0.099]
```

Nine lines, and they are the last two boxes of nearly every classifier in production.

##1. Conclusion
Softmax and cross-entropy are a matched pair, not two independent choices. Softmax answers *"how do I turn arbitrary scores into a distribution ?"* with the only construction that is positive, order-preserving and depends solely on score differences ; cross-entropy answers *"how wrong is that distribution ?"* with the information-theoretic surprise of the true answer, which is the same thing as the KL divergence to the truth. Chained together, the exponential and the logarithm cancel and leave the cleanest gradient in machine learning, $p - y$ : *push down what you predicted, push up what was true.* Hand that to [backpropagation]({{ site.baseurl }}/blog/machine-learning/backpropagation), let [gradient descent]({{ site.baseurl }}/blog/machine-learning/gradient-descent) take a small step, and repeat. [^1]

The name *softmax* itself dates back to Bridle {% cite Bridle1990 %}, who introduced it precisely so that a classifier's outputs could be read as probabilities and trained by maximum likelihood — which, once you take the logarithm and flip the sign, is exactly the cross-entropy of this post. Goodfellow, Bengio and Courville {% cite Goodfellow2016 %} give the full treatment.


References
----------
{% bibliography --cited %}

[^1]: The same pairing shows up under different names all over statistics and machine learning : with $K=2$ it is *logistic regression* trained by *log-loss*, which is also the default objective of the [gradient boosting]({{ site.baseurl }}/blog/machine-learning/xgboost) machinery, and in general it is nothing but *maximum likelihood estimation* for a categorical distribution.
