---
layout: post
title: The Attention Mechanism, Explained
date: 2024-07-12 12:00
comments: true
external-url:
categories: [Machine Learning]
tags: [machine_learning, deep_learning, nlp]
permalink: /blog/machine-learning/attention-mechanism
---


> Read this sentence : *"The cat sat on the mat because **it** was tired."* Who was tired ? You knew instantly it was the cat, not the mat, because your brain **paid attention** to the right earlier word. The **attention mechanism**, introduced for translation by Bahdanau {% cite Bahdanau2014 %} and then crowned as the single idea behind the Transformer {% cite Vaswani2017 %}, gives a neural network exactly this superpower : for every word, it decides which other words to look at, and how hard. This post builds attention from an everyday intuition up to the famous formula $\operatorname{softmax}\!\big(QK^{\top}/\sqrt{d_k}\big)V$, one small step at a time.

##1. The problem : words need context
A word on its own is ambiguous. *"Bank"* is a riverside or a place for money ; *"it"* points to something mentioned earlier ; *"apple"* could be a fruit or a company. To understand a word, a model must **mix in information from the other words around it**.

Older models (recurrent networks) read a sentence one word at a time, squeezing everything seen so far into a single memory vector. Like whispering a long story down a line of people, the beginning gets forgotten by the end. Attention throws that bottleneck away : it lets every word look **directly** at every other word, no matter how far apart, all at once.

<div class="definition">
	<b>Attention</b> is a mechanism that computes, for each element of a sequence, a new representation as a <b>weighted average of the other elements</b>, where the weights (how much to "attend") are learned from how <i>relevant</i> each element is to it.
</div>

##1. The big idea : looking things up
Here is the ELI5 picture. Imagine a tiny library search.

- You walk in with a **question** — *"something about a tired animal"*. That is your **query**.
- Every book on the shelf has a **label** describing what it is about — *"cats", "mats", "weather"*. Those are the **keys**.
- Every book also has actual **contents** inside. Those are the **values**.

To answer your question you compare it against each label, decide how well each matches, and then read *mostly* the best-matching books, skimming the rest. Attention is precisely this : compare a **query** against all **keys**, turn the match scores into percentages, and blend the **values** accordingly.

<div class="definition">
	Each element is turned into three vectors : a <b>query</b> $q$ ("what am I looking for ?"), a <b>key</b> $k$ ("what do I offer ?"), and a <b>value</b> $v$ ("what information do I carry ?"). These are produced by multiplying the element's embedding $x$ by three learned matrices, $q = W_{Q}\,x$, $k = W_{K}\,x$, $v = W_{V}\,x$.
</div>

##1. Attention for a single word, step by step
Let us attend from one query $q$ to a set of keys $k_{1}, \dots, k_{n}$ and values $v_{1}, \dots, v_{n}$. Four small steps.

**Step 1 — Score.** How relevant is each word to my query ? Measure it with a **dot product** [^1], which is large when two vectors point the same way :

$$ \begin{equation} s_{j} = q \cdot k_{j} = \sum_{d} q_{d}\, k_{j,d}. \label{eq:score} \end{equation} $$

*In plain words : a high score $s_j$ means "this word is a good match for what I'm looking for".*

**Step 2 — Scale.** Divide by $\sqrt{d_k}$ (with $d_k$ the vector length). We will justify this shortly ; for now, it just keeps the numbers from getting too big.

**Step 3 — Softmax.** Turn the scores into positive weights that add up to $1$ (percentages of attention), exactly the [softmax]({{ site.baseurl }}/blog/machine-learning/softmax-cross-entropy) that turns raw scores into a distribution :

$$ \begin{equation} \alpha_{j} = \frac{\exp(s_{j}/\sqrt{d_k})}{\sum_{\ell=1}^{n}\exp(s_{\ell}/\sqrt{d_k})}, \qquad \sum_{j}\alpha_{j}=1. \label{eq:softmax} \end{equation} $$

**Step 4 — Blend.** Read the values in those proportions :

$$ \begin{equation} \text{output} = \sum_{j=1}^{n}\alpha_{j}\, v_{j}. \label{eq:blend} \end{equation} $$

*In plain words : the answer is a mix of all the words' contents, but dominated by the ones that matched.* The animation below shows the query *"it"* scanning the sentence : the bars are the attention weights $\alpha_j$, and *"cat"* wins.

<figure><center>
<svg viewBox="0 0 640 300" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g1-word { font-size:15px; fill:#2b2b2b; } .g1-q { font-style:italic; fill:#b5651d; }
    .g1-bar  { fill:#4a6da7; transform-box:fill-box; transform-origin:bottom;
            animation:grow 4s ease-in-out infinite; }
    @keyframes grow { 0%{transform:scaleY(0)} 25%{transform:scaleY(1)} 85%{transform:scaleY(1)} 100%{transform:scaleY(0)} }
    .g1-cap { font-size:12px; fill:#7a7a7a; }
    .g1-spot{ fill:#b5651d; opacity:.12; animation:sweep 4s ease-in-out infinite; }
    @keyframes sweep { 0%{opacity:0} 30%{opacity:.16} 100%{opacity:.16} }
    .g1-pct { font-size:10.5px; fill:#4a6da7; }
  </style>
  <text class="g1-word g1-q" x="320" y="30" text-anchor="middle">query : "it"  →  who / what ?</text>
  <!-- 7 words, bars proportional to attention weight -->
  <!-- x positions -->
  <!-- The:0.03 cat:0.55 sat:0.10 because:0.05 it:0.02 was:0.05 tired:0.20 -->
  <!-- baseline y=230; max bar height 150 -->
  <!-- cat spotlight -->
  <rect class="g1-spot" x="112" y="70" width="70" height="200" rx="6"/>
  <g text-anchor="middle">
    <rect class="g1-bar" x="36"  y="225" width="30" height="5"   style="animation-delay:.0s"/><text class="g1-word" x="51"  y="252">The</text>   <text class="g1-pct" x="51"  y="266">3%</text>
    <rect class="g1-bar" x="118" y="80"  width="58" height="150" style="animation-delay:.1s"/><text class="g1-word" x="147" y="252">cat</text>   <text class="g1-pct" x="147" y="266">55%</text>
    <rect class="g1-bar" x="196" y="200" width="30" height="30"  style="animation-delay:.2s"/><text class="g1-word" x="211" y="252">sat</text>   <text class="g1-pct" x="211" y="266">10%</text>
    <rect class="g1-bar" x="262" y="215" width="46" height="15"  style="animation-delay:.3s"/><text class="g1-word" x="285" y="252">because</text><text class="g1-pct" x="285" y="266">5%</text>
    <rect class="g1-bar" x="336" y="222" width="24" height="8"   style="animation-delay:.4s"/><text class="g1-word g1-q" x="348" y="252">it</text>    <text class="g1-pct" x="348" y="266">2%</text>
    <rect class="g1-bar" x="382" y="215" width="34" height="15"  style="animation-delay:.5s"/><text class="g1-word" x="399" y="252">was</text>   <text class="g1-pct" x="399" y="266">5%</text>
    <rect class="g1-bar" x="438" y="170" width="46" height="60"  style="animation-delay:.6s"/><text class="g1-word" x="461" y="252">tired</text> <text class="g1-pct" x="461" y="266">20%</text>
  </g>
  <line x1="24" y1="230" x2="500" y2="230" stroke="#ccc" stroke-width="1"/>
  <text class="g1-cap" x="320" y="290" text-anchor="middle">attention weights αⱼ (they sum to 100%) — "it" attends mostly to "cat"</text>
</svg>
<figcaption><b>Figure 1 -</b> Attention from the word <i>"it"</i>. Each bar is the learned weight $\alpha_j$ from $\eqref{eq:softmax}$. The output for <i>"it"</i> becomes mostly the <i>value</i> of <i>"cat"</i>, so the model now "knows" what <i>it</i> refers to.</figcaption>
</center></figure>

##1. All queries at once : the matrix formula
A sentence has many words, and each is a query. Instead of looping, we stack the query, key and value vectors as rows of matrices $Q$, $K$, $V$ and do it all with two matrix multiplications. This is the celebrated **scaled dot-product attention** {% cite Vaswani2017 %} :

<div class="theorem" text="Scaled dot-product attention">
	$$ \operatorname{Attention}(Q, K, V) = \operatorname{softmax}\!\left( \frac{Q K^{\top}}{\sqrt{d_k}} \right) V. $$
</div>

Let us read it left to right, because every piece is one of the four steps above.

- $Q K^{\top}$ is the table of **all** scores at once : entry $(i,j)$ is query $i$ dotted with key $j$, exactly $\eqref{eq:score}$.
- Dividing by $\sqrt{d_k}$ is the **scaling** of Step 2.
- $\operatorname{softmax}$ (applied row by row) turns each row into attention weights that sum to $1$, exactly $\eqref{eq:softmax}$.
- Multiplying by $V$ **blends the values**, exactly $\eqref{eq:blend}$, but for every word simultaneously.

<figure><center>
<svg viewBox="0 0 640 250" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g2-box { rx:8; stroke-width:1.6; } 
    .g2-bq{ fill:#f6f7fb; stroke:#4a6da7; } .g2-bk{ fill:#f6f7fb; stroke:#4a6da7; }
    .g2-bv{ fill:#f3faf4; stroke:#5aa06a; } .g2-bs{ fill:#faf6f2; stroke:#b5651d; }
    .g2-bo{ fill:#fff; stroke:#2b2b2b; }
    .g2-t { font-size:14px; fill:#2b2b2b; text-anchor:middle; } .g2-op{ font-size:20px; fill:#7a7a7a; text-anchor:middle; }
    .g2-sm{ font-size:11px; fill:#7a7a7a; text-anchor:middle; }
    .g2-arrow{ stroke:#b5651d; stroke-width:1.8; fill:none; stroke-dasharray:5 6; animation:aflow 1.8s linear infinite; }
    @keyframes aflow { to { stroke-dashoffset:-44; } }
  </style>
  <!-- Q x K^T -->
  <rect class="g2-box g2-bq" x="20"  y="60" width="60" height="60"/><text class="g2-t" x="50"  y="96">Q</text>
  <text class="g2-op" x="95" y="96">×</text>
  <rect class="g2-box g2-bk" x="110" y="60" width="60" height="60"/><text class="g2-t" x="140" y="96">Kᵀ</text>
  <path class="g2-arrow" d="M175,90 L215,90"/>
  <!-- scores -->
  <rect class="g2-box g2-bs" x="220" y="55" width="70" height="70"/><text class="g2-t" x="255" y="88">QKᵀ</text><text class="g2-sm" x="255" y="104">/ √dₖ</text>
  <path class="g2-arrow" d="M295,90 L335,90"/>
  <!-- softmax -->
  <rect class="g2-box g2-bs" x="340" y="55" width="90" height="70"/><text class="g2-t" x="385" y="95">softmax</text>
  <text class="g2-sm" x="385" y="145">attention weights A</text>
  <path class="g2-arrow" d="M435,90 L475,90"/>
  <text class="g2-op" x="497" y="96">×</text>
  <!-- V -->
  <rect class="g2-box g2-bv" x="515" y="60" width="55" height="60"/><text class="g2-t" x="542" y="96">V</text>
  <!-- output -->
  <path class="g2-arrow" d="M542,125 L542,165"/>
  <rect class="g2-box g2-bo" x="500" y="170" width="85" height="55"/><text class="g2-t" x="542" y="203">output</text>
  <text class="g2-sm" x="300" y="185">scores → weights (each row sums to 1)</text>
</svg>
<figcaption><b>Figure 2 -</b> The pipeline of scaled dot-product attention. Scores are computed ($QK^{\top}$), scaled, softmaxed into a weight table $A$, and used to average the values $V$.</figcaption>
</center></figure>

<div class="example">
	Why divide by $\sqrt{d_k}$ ? If queries and keys have independent, unit-variance entries, their dot product $\eqref{eq:score}$ is a sum of $d_k$ random terms, so its variance grows like $d_k$. For large $d_k$ the scores become huge, the softmax saturates (one weight $\approx 1$, the rest $\approx 0$), and gradients vanish. Dividing by $\sqrt{d_k}$ rescales the variance back to $\approx 1$, keeping the softmax in its sensitive, learnable range.
</div>

##1. Self-attention : a sentence looking at itself
When the queries, keys and values all come from the **same** sentence ($Q$, $K$, $V$ are three projections of the same words), we call it **self-attention**. Every word refreshes itself by mixing in the words most relevant to it. The full weight table $A = \operatorname{softmax}(QK^{\top}/\sqrt{d_k})$ is an $n \times n$ heatmap : row $i$ says where word $i$ looks.

<figure><center>
<svg viewBox="0 0 640 300" width="100%" style="max-width:560px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .g3-hl { font-size:12.5px; fill:#2b2b2b; }
    .g3-cellh { stroke:#fff; stroke-width:2; }
    .g3-rowscan { fill:#b5651d; opacity:0; animation:rowsweep 5s steps(1) infinite; }
    @keyframes rowsweep { 0%,100%{opacity:0} }
    .g3-lab{ font-size:11px; fill:#7a7a7a; }
  </style>
  <!-- 5 words -->
  <!-- labels top (keys) and left (queries) -->
  <text class="g3-lab" x="330" y="24" text-anchor="middle">keys  (looked at)  →</text>
  <text class="g3-lab" x="20" y="160" text-anchor="middle" transform="rotate(-90 20 160)">queries  (doing the looking)  →</text>
  <g class="g3-hl" text-anchor="middle">
    <text x="120" y="52">The</text><text x="190" y="52">cat</text><text x="260" y="52">sat</text><text x="330" y="52">on</text><text x="400" y="52">mat</text>
  </g>
  <g class="g3-hl" text-anchor="end">
    <text x="86" y="90">The</text><text x="86" y="140">cat</text><text x="86" y="190">sat</text><text x="86" y="240">on</text><text x="86" y="290">mat</text>
  </g>
  <!-- heat cells: opacity encodes weight. rows sum ~1 -->
  <g transform="translate(95,64)">
    <!-- row The -->
    <rect class="g3-cellh" x="0"   y="0"  width="50" height="40" fill="#4a6da7" opacity="0.75"/>
    <rect class="g3-cellh" x="70"  y="0"  width="50" height="40" fill="#4a6da7" opacity="0.10"/>
    <rect class="g3-cellh" x="140" y="0"  width="50" height="40" fill="#4a6da7" opacity="0.06"/>
    <rect class="g3-cellh" x="210" y="0"  width="50" height="40" fill="#4a6da7" opacity="0.05"/>
    <rect class="g3-cellh" x="280" y="0"  width="50" height="40" fill="#4a6da7" opacity="0.04"/>
    <!-- row cat -->
    <rect class="g3-cellh" x="0"   y="50" width="50" height="40" fill="#4a6da7" opacity="0.08"/>
    <rect class="g3-cellh" x="70"  y="50" width="50" height="40" fill="#4a6da7" opacity="0.62"/>
    <rect class="g3-cellh" x="140" y="50" width="50" height="40" fill="#4a6da7" opacity="0.20"/>
    <rect class="g3-cellh" x="210" y="50" width="50" height="40" fill="#4a6da7" opacity="0.05"/>
    <rect class="g3-cellh" x="280" y="50" width="50" height="40" fill="#4a6da7" opacity="0.05"/>
    <!-- row sat -->
    <rect class="g3-cellh" x="0"   y="100" width="50" height="40" fill="#4a6da7" opacity="0.06"/>
    <rect class="g3-cellh" x="70"  y="100" width="50" height="40" fill="#4a6da7" opacity="0.30"/>
    <rect class="g3-cellh" x="140" y="100" width="50" height="40" fill="#4a6da7" opacity="0.44"/>
    <rect class="g3-cellh" x="210" y="100" width="50" height="40" fill="#4a6da7" opacity="0.12"/>
    <rect class="g3-cellh" x="280" y="100" width="50" height="40" fill="#4a6da7" opacity="0.08"/>
    <!-- row on -->
    <rect class="g3-cellh" x="0"   y="150" width="50" height="40" fill="#4a6da7" opacity="0.05"/>
    <rect class="g3-cellh" x="70"  y="150" width="50" height="40" fill="#4a6da7" opacity="0.08"/>
    <rect class="g3-cellh" x="140" y="150" width="50" height="40" fill="#4a6da7" opacity="0.22"/>
    <rect class="g3-cellh" x="210" y="150" width="50" height="40" fill="#4a6da7" opacity="0.45"/>
    <rect class="g3-cellh" x="280" y="150" width="50" height="40" fill="#4a6da7" opacity="0.20"/>
    <!-- row mat -->
    <rect class="g3-cellh" x="0"   y="200" width="50" height="40" fill="#4a6da7" opacity="0.04"/>
    <rect class="g3-cellh" x="70"  y="200" width="50" height="40" fill="#4a6da7" opacity="0.18"/>
    <rect class="g3-cellh" x="140" y="200" width="50" height="40" fill="#4a6da7" opacity="0.10"/>
    <rect class="g3-cellh" x="210" y="200" width="50" height="40" fill="#4a6da7" opacity="0.20"/>
    <rect class="g3-cellh" x="280" y="200" width="50" height="40" fill="#4a6da7" opacity="0.48"/>
    <!-- animated highlight sweeping down the rows -->
    <rect x="-4" y="-4" width="338" height="48" fill="none" stroke="#b5651d" stroke-width="2.4" rx="3">
      <animate attributeName="y" values="-4;46;96;146;196;-4" dur="5s" repeatCount="indefinite"/>
    </rect>
  </g>
  <text class="g3-lab" x="330" y="288" text-anchor="middle">darker = more attention · the moving box scans one query (row) at a time</text>
</svg>
<figcaption><b>Figure 3 -</b> A self-attention heatmap for <i>"The cat sat on mat"</i>. Each row is one word's attention distribution over all words. Notice the diagonal (words attend to themselves) plus meaningful off-diagonal links like <i>sat</i>→<i>cat</i>.</figcaption>
</center></figure>

##1. Many spotlights : multi-head attention
One attention pattern can only capture one kind of relationship. Real language has several at once : *who did what* (subject–verb), *what modifies what* (adjective–noun), *what refers to what* (pronoun–antecedent). So Transformers run several attentions in parallel, called **heads**, each with its own $W_Q, W_K, W_V$, and then concatenate the results :

$$ \begin{equation} \operatorname{MultiHead}(Q,K,V) = \operatorname{Concat}(\text{head}_1, \dots, \text{head}_h)\, W_{O}, \quad \text{head}_i = \operatorname{Attention}(QW_Q^i, KW_K^i, VW_V^i). \label{eq:multihead} \end{equation} $$

*In plain words : each head is a different pair of "reading glasses". One head might track grammar, another long-range references ; stacking their views gives a richer representation.*

##1. Why attention changed everything
Two properties made attention the backbone of modern AI.

- **Parallelism.** Every word is processed at the same time, not one after another, so training uses modern hardware fully. This is the practical reason it dethroned recurrent networks.
- **Direct long-range links.** Any two words are one step apart, so a pronoun can look back at a noun fifty words earlier with no signal decay.

Attention is the *engine*. To build a full model, we stack it into a repeating block, add a way to encode word order, and sprinkle in some plumbing. That assembled machine is the **Transformer**, the subject of a [companion post]({{ site.baseurl }}/blog/machine-learning/transformers). And why can a stack of these blocks represent essentially any function we need ? That question is answered by the [Universal Approximation Theorem]({{ site.baseurl }}/blog/mathematics/universal-approximation-theorem).

##1. Conclusion
Attention is, at heart, a *soft, differentiable lookup table* : compare a query to keys, softmax the matches into weights, and average the values. Everything else, the scaling by $\sqrt{d_k}$, self-attention, multiple heads, is refinement on that one idea. It replaced the fragile "remember everything in one vector" strategy with a simple instruction that a network can learn : *for each word, look at the words that matter.* From that sentence-sized intuition grew the models that now write, translate and reason.


References
----------
{% bibliography --cited %}

[^1]: The dot-product form used here is sometimes called *multiplicative* attention. Bahdanau's original 2014 formulation used an *additive* variant with a small feed-forward network to score query–key pairs ; the two perform comparably, but the multiplicative form is far faster as a matrix product.
