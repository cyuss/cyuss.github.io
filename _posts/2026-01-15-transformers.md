---
layout: post
title: The Transformer, Explained
date: 2026-01-15 12:00
comments: true
external-url:
categories: [Machine Learning]
tags: [machine_learning, deep_learning, nlp]
permalink: /blog/machine-learning/transformers
---


> If [attention]({{ site.baseurl }}/blog/machine-learning/attention-mechanism) is an *engine*, the **Transformer** is the whole *car* built around it. Introduced in the 2017 paper whose title says it all, *"Attention Is All You Need"* {% cite Vaswani2017 %}, it threw away the sequential recurrence that machine translation had relied on and replaced it with stacks of attention, processed all at once. That single architectural bet powers essentially every large language model you have heard of, from BERT to the GPT family. This post assembles the Transformer piece by piece, keeping each part ELI5-simple while writing down the mathematics that makes it tick. If you have not met attention yet, read the [companion post]({{ site.baseurl }}/blog/machine-learning/attention-mechanism) first, everything here is built on it.

##1. The one-sentence idea
A Transformer takes a sequence of words, and repeatedly lets every word **look at every other word** (attention) and then **think for itself** (a small neural network), over and over, until the representations are rich enough to do the task, translate, summarise, or predict the next word.

That is genuinely most of it. The rest of this post is about the plumbing that makes this idea trainable : how to tell the model the *order* of the words, how to keep gradients healthy across a deep stack, and how the two halves (encoder and decoder) cooperate.

##1. Words in, numbers in : embeddings and order
A model cannot chew on letters, so each token (word or sub-word) is first mapped to a vector, its **embedding**. A sentence of $n$ tokens becomes a matrix $X \in \Bbb{R}^{n \times d}$ of $n$ rows, each of dimension $d$.

But attention has a curious blind spot : it treats its input as a **bag** of words. Shuffle the sentence and the attention maths gives the same answer, because a weighted average does not care about order. *"Dog bites man"* and *"man bites dog"* would look identical, which is clearly unacceptable. The fix is to **add a position signal** to every embedding.

The original Transformer uses fixed **sinusoidal positional encodings** : for position $pos$ and dimension $i$,

$$ \begin{equation} \begin{aligned} PE(pos, 2i)   &= \sin\!\left( \frac{pos}{10000^{\,2i/d}} \right), \\ PE(pos, 2i+1) &= \cos\!\left( \frac{pos}{10000^{\,2i/d}} \right). \end{aligned} \label{eq:posenc} \end{equation} $$

*In plain words : each position gets a unique "fingerprint" made of sine waves of many different wavelengths, fast wiggles for fine position, slow waves for coarse position, just like the hour, minute and second hands of a clock together pin down a unique time.* Because sines and cosines shift predictably, the model can also learn **relative** offsets ("three words later") easily.

<figure><center>
<svg viewBox="0 0 640 260" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .axp{ stroke:#bbb; stroke-width:1; }
    .w1{ fill:none; stroke:#4a6da7; stroke-width:2; }
    .w2{ fill:none; stroke:#5aa06a; stroke-width:2; }
    .w3{ fill:none; stroke:#b5651d; stroke-width:2; }
    .scan{ stroke:#2b2b2b; stroke-width:1.4; stroke-dasharray:3 3; }
    .dot{ r:4.5; }
    .lab{ font-size:12px; fill:#7a7a7a; } .k{ font-size:12px; }
  </style>
  <line class="axp" x1="40" y1="130" x2="600" y2="130"/>
  <text class="lab" x="320" y="250" text-anchor="middle">position  →   (a vertical slice = that position's unique fingerprint)</text>
  <!-- three sine waves of different wavelengths over x 40..600 -->
  <!-- fast -->
  <path class="w3" d="M40,130 Q55,70 70,130 T100,130 T130,130 T160,130 T190,130 T220,130 T250,130 T280,130 T310,130 T340,130 T370,130 T400,130 T430,130 T460,130 T490,130 T520,130 T550,130 T580,130 T600,130"/>
  <!-- medium (green) : use a real sine sampled -->
  <path class="w2" d="M40,130 C90,50 140,50 190,130 C240,210 290,210 340,130 C390,50 440,50 490,130 C540,210 590,210 620,130"/>
  <!-- slow (blue) -->
  <path class="w1" d="M40,130 C140,40 240,40 340,130 C440,220 540,220 620,130"/>
  <!-- animated scan line -->
  <line class="scan" x1="40" y1="20" x2="40" y2="235">
    <animate attributeName="x1" values="40;600;40" dur="6s" repeatCount="indefinite"/>
    <animate attributeName="x2" values="40;600;40" dur="6s" repeatCount="indefinite"/>
  </line>
  <circle class="dot" fill="#4a6da7"><animateMotion path="M40,130 C140,40 240,40 340,130 C440,220 540,220 620,130" dur="6s" repeatCount="indefinite"/></circle>
  <text class="k" x="70"  y="40" fill="#b5651d">dim i : fast wave</text>
  <text class="k" x="360" y="40" fill="#5aa06a">dim j : medium</text>
  <text class="k" x="360" y="220" fill="#4a6da7">dim k : slow wave</text>
</svg>
<figcaption><b>Figure 1 -</b> Sinusoidal positional encodings $\eqref{eq:posenc}$. Different embedding dimensions oscillate at different wavelengths ; reading one vertical slice gives a position its unique code. The scanning line sweeps through positions.</figcaption>
</center></figure>

##1. The repeating block
The heart of the Transformer is one block, stacked $N$ times (the paper uses $N = 6$) [^1]. A block does two things in sequence : **mix words** with multi-head self-attention, then **process each word** with a small feed-forward network. Two extra tricks keep such a deep stack trainable.

- **Residual connections.** Each sub-layer computes $x + \operatorname{SubLayer}(x)$, not just $\operatorname{SubLayer}(x)$. The input is added back, so information (and gradient) can always flow straight through. ELI5 : *the block only has to learn the "edit", not re-copy everything it received.*
- **Layer normalisation.** After each add, the values are renormalised to a stable scale, which keeps training smooth.

Putting it together, one encoder block computes

$$ \begin{equation} \begin{aligned} Z &= \operatorname{LayerNorm}\big(X + \operatorname{MultiHead}(X, X, X)\big), \\ \operatorname{Block}(X) &= \operatorname{LayerNorm}\big(Z + \operatorname{FFN}(Z)\big), \end{aligned} \label{eq:block} \end{equation} $$

where the position-wise feed-forward network is just two linear layers with a non-linearity, applied to each word identically :

$$ \begin{equation} \operatorname{FFN}(z) = \max(0,\; z W_1 + b_1)\, W_2 + b_2. \label{eq:ffn} \end{equation} $$

<figure><center>
<svg viewBox="0 0 480 430" width="100%" style="max-width:380px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .flow{ stroke:#4a6da7; stroke-width:2.2; fill:none; marker-end:url(#ah);
           stroke-dasharray:6 7; animation:tflow 2.2s linear infinite; }
    @keyframes tflow { to { stroke-dashoffset:-52; } }
    .res { stroke:#b5651d; stroke-width:1.8; fill:none; stroke-dasharray:4 4; }
    .b   { rx:9; stroke-width:1.7; }
    .att { fill:#f6f7fb; stroke:#4a6da7; } .ffn{ fill:#f3faf4; stroke:#5aa06a; }
    .an  { fill:#faf6f2; stroke:#b5651d; } .io{ fill:#fff; stroke:#2b2b2b; }
    .t   { font-size:13px; fill:#2b2b2b; text-anchor:middle; }
    .rt  { font-size:11px; fill:#b5651d; }
  </style>
  <defs><marker id="ah" markerWidth="9" markerHeight="9" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="#4a6da7"/></marker></defs>
  <!-- input -->
  <rect class="b io" x="175" y="388" width="130" height="34"/><text class="t" x="240" y="410">input  X</text>
  <path class="flow" d="M240,388 L240,352"/>
  <!-- attention -->
  <rect class="b att" x="120" y="300" width="240" height="48"/><text class="t" x="240" y="329">Multi-Head Self-Attention</text>
  <path class="flow" d="M240,300 L240,270"/>
  <!-- add & norm 1 -->
  <rect class="b an" x="150" y="234" width="180" height="34"/><text class="t" x="240" y="256">Add &amp; Norm</text>
  <path class="flow" d="M240,234 L240,204"/>
  <!-- ffn -->
  <rect class="b ffn" x="140" y="156" width="200" height="46"/><text class="t" x="240" y="184">Feed-Forward</text>
  <path class="flow" d="M240,156 L240,126"/>
  <!-- add & norm 2 -->
  <rect class="b an" x="150" y="90" width="180" height="34"/><text class="t" x="240" y="112">Add &amp; Norm</text>
  <path class="flow" d="M240,90 L240,58"/>
  <!-- output -->
  <rect class="b io" x="165" y="20" width="150" height="34"/><text class="t" x="240" y="42">block output</text>
  <!-- residual arrows -->
  <path class="res" d="M120,370 C70,360 70,270 145,251"/>
  <path class="res" d="M120,286 C58,270 58,120 145,107"/>
  <text class="rt" x="40" y="315" transform="rotate(-90 40 315)">residual +</text>
  <text class="rt" x="30" y="200" transform="rotate(-90 30 200)">residual +</text>
</svg>
<figcaption><b>Figure 2 -</b> One Transformer block $\eqref{eq:block}$. Data flows bottom to top (blue) : self-attention, then a feed-forward network, each wrapped in a residual "skip" (orange) and layer normalisation. Stack this block $N$ times to get an encoder.</figcaption>
</center></figure>

##1. Encoder and decoder
The full translation model has **two towers** (Figure 3). ELI5 : the **encoder** *reads* the source sentence and builds an understanding of it ; the **decoder** *writes* the translation one word at a time, consulting that understanding.

The decoder block has one extra ingredient and one twist :

- **Cross-attention.** A middle attention layer where the *queries* come from the translation-so-far but the *keys and values* come from the encoder's output. This is how the decoder "looks back" at the source, exactly the alignment that attention was originally invented for {% cite Bahdanau2014 %}.
- **Masked self-attention.** When writing word $t$, the decoder must not peek at words $t+1, t+2, \dots$ (they do not exist yet at generation time). We enforce this by setting the forbidden scores to $-\infty$ before the softmax, so their weight becomes $0$ :

$$ \begin{equation} \big(\text{mask}\big)_{ij} = \begin{cases} 0 & j \le i \quad (\text{allowed : look at past and present}) \\ -\infty & j > i \quad (\text{forbidden : the future}) \end{cases} \label{eq:mask} \end{equation} $$

<figure><center>
<svg viewBox="0 0 640 380" width="100%" style="max-width:560px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .stack{ rx:8; stroke-width:1.6; }
    .enc{ fill:#f6f7fb; stroke:#4a6da7; } .dec{ fill:#f3faf4; stroke:#5aa06a; }
    .io{ fill:#fff; stroke:#2b2b2b; rx:8; stroke-width:1.6; }
    .t{ font-size:12.5px; fill:#2b2b2b; text-anchor:middle; } .nx{ font-size:11px; fill:#7a7a7a; text-anchor:middle; }
    .up{ stroke:#4a6da7; stroke-width:2; fill:none; marker-end:url(#a2); stroke-dasharray:6 6; animation:uf 2s linear infinite;}
    .cross{ stroke:#b5651d; stroke-width:2; fill:none; marker-end:url(#a3); stroke-dasharray:5 5; animation:uf 2s linear infinite;}
    @keyframes uf{ to{ stroke-dashoffset:-48; } }
  </style>
  <defs>
    <marker id="a2" markerWidth="9" markerHeight="9" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="#4a6da7"/></marker>
    <marker id="a3" markerWidth="9" markerHeight="9" refX="6" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="#b5651d"/></marker>
  </defs>
  <!-- ENCODER -->
  <text class="nx" x="150" y="26">ENCODER  (reads source)</text>
  <rect class="io" x="90" y="320" width="120" height="34"/><text class="t" x="150" y="342">"le chat"</text>
  <path class="up" d="M150,320 L150,292"/>
  <rect class="stack enc" x="80" y="230" width="140" height="58"/><text class="t" x="150" y="256">Self-Attention</text><text class="t" x="150" y="274">+ Feed-Forward</text>
  <text class="nx" x="150" y="305">× N blocks</text>
  <path class="up" d="M150,230 L150,150"/>
  <rect class="stack enc" x="80" y="92" width="140" height="58" opacity=".5"/><text class="t" x="150" y="126">encoded source</text>
  <!-- DECODER -->
  <text class="nx" x="470" y="26">DECODER  (writes target)</text>
  <rect class="io" x="410" y="320" width="150" height="34"/><text class="t" x="485" y="342">"the ___" (so far)</text>
  <path class="up" d="M485,320 L485,296"/>
  <rect class="stack dec" x="395" y="236" width="180" height="30"/><text class="t" x="485" y="256">Masked Self-Attention</text>
  <path class="up" d="M485,236 L485,214"/>
  <rect class="stack dec" x="395" y="176" width="180" height="34"/><text class="t" x="485" y="198">Cross-Attention</text>
  <path class="up" d="M485,176 L485,150"/>
  <rect class="stack dec" x="395" y="112" width="180" height="34"/><text class="t" x="485" y="134">Feed-Forward</text>
  <text class="nx" x="485" y="300">× N blocks</text>
  <path class="up" d="M485,112 L485,84"/>
  <rect class="io" x="410" y="46" width="150" height="34"/><text class="t" x="485" y="68">next word : "cat"</text>
  <!-- cross attention link encoder -> decoder -->
  <path class="cross" d="M220,121 C300,121 320,193 388,193"/>
  <text class="nx" x="300" y="176" fill="#b5651d">keys / values</text>
</svg>
<figcaption><b>Figure 3 -</b> The encoder–decoder Transformer. The encoder (blue) turns the source into a set of key/value vectors ; the decoder (green) generates the target word by word, using masked self-attention over what it has written and cross-attention (orange) into the encoder's output.</figcaption>
</center></figure>

##1. Generating text, one word at a time
At the top of the decoder, a linear layer plus a softmax turns the final vector into a probability distribution over the whole vocabulary :

$$ \begin{equation} p(\text{next word}) = \operatorname{softmax}(h\, W_{\text{vocab}} + b). \label{eq:head} \end{equation} $$

The model picks a word from that distribution, appends it to the sentence, and runs again to produce the next, this is **autoregressive** generation (the `temperature` knob you set when sampling is the one described in the [softmax and cross-entropy]({{ site.baseurl }}/blog/machine-learning/softmax-cross-entropy) post). ELI5 : *it writes like you texting with predictive keyboard, one suggestion at a time, each new word feeding back in as context.* The masking $\eqref{eq:mask}$ is what makes training match this behaviour : during training the model sees the whole target sentence but is forbidden from looking ahead, so it learns to predict each word from only the words before it.

##1. Why Transformers took over
- **Full parallelism.** Unlike recurrent models that must process word 1 before word 2, a Transformer handles all positions simultaneously, turning training into big matrix multiplications that GPUs devour. This is the pragmatic reason they scaled.
- **Constant path length.** Any two tokens interact directly through attention, so long-range dependencies are as easy as short ones.
- **It just keeps scaling.** Stack more blocks, widen the vectors, feed more data, and performance keeps improving, the empirical observation that launched the era of large language models. BERT {% cite Devlin2019 %} kept only the encoder (for understanding) ; the GPT family kept only the decoder (for generation) ; both are the same block from Figure 2, repeated.

Why should stacking these blocks be able to represent the staggering variety of functions language requires ? The reassuring theoretical backdrop is the [Universal Approximation Theorem]({{ site.baseurl }}/blog/mathematics/universal-approximation-theorem) : the feed-forward sub-layers alone are universal approximators, and attention adds the ability to route information between positions. Representational power was never the worry ; as that post stresses, the real magic is that these models can be *trained* at all.

##1. Conclusion
Strip away the vocabulary and the Transformer is short to describe : embed the tokens, add a positional fingerprint, then repeat *"let every word look at the others, then let every word think"* a handful of times, with residual skips and normalisation to keep the deep stack healthy. Two towers, an encoder that reads and a decoder that writes, cooperate through cross-attention, and a masked softmax lets the whole thing generate text one word at a time. No recurrence, no convolution, [attention]({{ site.baseurl }}/blog/machine-learning/attention-mechanism) really was, more or less, all we needed.


References
----------
{% bibliography --cited %}

[^1]: The paper's base model uses $N = 6$ blocks, model dimension $d = 512$, $h = 8$ attention heads, and a feed-forward inner size of $2048$. Modern large language models keep the same block but scale these numbers by orders of magnitude.
