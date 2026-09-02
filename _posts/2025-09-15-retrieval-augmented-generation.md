---
layout: post
title: Introduction to Retrieval-Augmented Generation
date: 2025-09-15 12:00
comments: true
external-url:
categories: [Machine Learning]
tags: [machine_learning, nlp, information_retrieval, llm]
permalink: /blog/machine-learning/retrieval-augmented-generation
---


> A language model is a closed-book student. It sat one enormous exam, memorised everything it could into its weights, and now answers from memory alone — confidently, fluently, and sometimes completely wrong, because it cannot tell the difference between remembering and inventing. **Retrieval-Augmented Generation** turns that into an *open-book* exam : before answering, the model is handed the three pages of your documentation that actually contain the answer. That single change fixes freshness, fixes attribution, and dramatically reduces invention — and it turns out that almost all the engineering, and almost all the failure, lives in the unglamorous half : *finding the right three pages*. This post builds the whole pipeline from first principles, with the geometry, the ranking formulas, and the honest limits.

##1. Why a model needs a library
A [Transformer]({{ site.baseurl }}/blog/machine-learning/transformers) stores what it learned in its weights. That **parametric knowledge** has three structural problems, and none of them is a bug you can train away :

- **It is frozen.** The weights encode the world as of the training cut-off. Yesterday's incident report, this morning's price change, your company's internal wiki — none of it is in there, and none of it ever will be.
- **It is unattributable.** The model cannot point at *where* it learned something, because the fact is not stored anywhere in particular : it is smeared across billions of parameters. "Cite your source" is not a request a purely parametric model can honour.
- **It is confidently lossy.** Training compresses an enormous corpus into a fixed number of weights. Compression loses detail, and when the model reconstructs a half-remembered detail it produces something *fluent* rather than something *true*. That is the mechanism behind hallucination.

The instinctive fix — fine-tune on your documents — addresses none of them properly. Fine-tuning is expensive, must be redone whenever the documents change, still gives no citations, and teaches *style* far more reliably than it teaches *facts*.

The other fix is embarrassingly simple : **do not make the model remember, let it look things up.** Put the documents in a searchable store ; when a question arrives, find the passages most likely to contain the answer ; paste them into the prompt ; ask the model to answer *from those passages and cite them*. That is Retrieval-Augmented Generation, introduced under that name by Lewis *et al.* {% cite Lewis2020 %}.

<div class="definition">
	A <b>RAG system</b> is a pair of stages. A <b>retriever</b> $R$ maps a question $q$ to a small ordered set of passages drawn from a corpus $\mathcal{D}$,
	$$ R(q) = (c_1, \dots, c_k) \subset \mathcal{D}, \qquad k \ll \lvert \mathcal{D} \rvert, $$
	and a <b>generator</b> $G$ (the language model) produces the answer conditioned on the question <i>and</i> those passages,
	$$ a = G\big(q,\; R(q)\big). $$
</div>

*In plain words : the retriever is a librarian who fetches a handful of pages, and the generator is a reader who answers using only those pages. Every property people like about RAG — freshness, citations, less invention — comes from the fact that the answer is conditioned on text the system can point at.*

<figure><center>
<svg viewBox="0 0 640 302" width="100%" style="max-width:625px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .bx{ fill:#f6f7fb; stroke:#4a6da7; stroke-width:1.5; }
    .ix{ fill:#f3faf4; stroke:#5aa06a; stroke-width:1.8; }
    .bt{ font-size:13px; fill:#2b2b2b; text-anchor:middle; }
    .bs{ font-size:9.5px; fill:#7a7a7a; text-anchor:middle; }
    .ln{ stroke:#4a6da7; stroke-width:1.8; fill:none; }
    .rt{ stroke:#5aa06a; stroke-linejoin:round; }
    .sw{ fill:#b5651d; fill-opacity:.10; stroke:#b5651d; stroke-width:2.2; }
    .lane{ font-size:11.5px; fill:#7a7a7a; font-style:italic; }
    .on{ fill:#b5651d; }
  </style>
  <defs>
    <marker id="am" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#4a6da7"/></marker>
    <marker id="ao" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#b5651d"/></marker>
    <marker id="ag" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#5aa06a"/></marker>
  </defs>
  <path class="ln" d="M148.0,63.0 L169.0,63.0" marker-end="url(#am)"/>
  <path class="ln" d="M272.0,63.0 L293.0,63.0" marker-end="url(#am)"/>
  <path class="ln" d="M396.0,63.0 L417.0,63.0" marker-end="url(#am)"/>
  <path class="ln" d="M120.0,246.0 L137.0,246.0" marker-end="url(#am)"/>
  <path class="ln" d="M232.0,246.0 L249.0,246.0" marker-end="url(#am)"/>
  <path class="ln" d="M360.0,246.0 L377.0,246.0" marker-end="url(#am)"/>
  <path class="ln" d="M488.0,246.0 L505.0,246.0" marker-end="url(#am)"/>
  <path class="ln rt" d="M499,90 L499,158 L308,158 L308,217" marker-end="url(#ag)"/>
  <rect class="bx" x="36" y="42" width="112" height="42" rx="5"/>
  <text class="bt" x="92" y="62.0">documents</text>
  <text class="bs" x="92" y="76.0">your corpus</text>
  <rect class="bx" x="176" y="42" width="96" height="42" rx="5"/>
  <text class="bt" x="224" y="62.0">chunks</text>
  <text class="bs" x="224" y="76.0">split + overlap</text>
  <rect class="bx" x="300" y="42" width="96" height="42" rx="5"/>
  <text class="bt" x="348" y="62.0">embed</text>
  <text class="bs" x="348" y="76.0">→ vectors</text>
  <rect class="bx ix" x="424" y="36" width="150" height="54" rx="5"/>
  <text class="bt" x="499" y="62.0">vector index</text>
  <text class="bs" x="499" y="76.0">ANN structure</text>
  <rect class="bx" x="16" y="224" width="104" height="44" rx="5"/>
  <text class="bt" x="68" y="245.0">question</text>
  <text class="bs" x="68" y="259.0">the user asks</text>
  <rect class="bx" x="144" y="224" width="88" height="44" rx="5"/>
  <text class="bt" x="188" y="245.0">embed</text>
  <text class="bs" x="188" y="259.0">→ vector q</text>
  <rect class="bx" x="256" y="224" width="104" height="44" rx="5"/>
  <text class="bt" x="308" y="245.0">retrieve</text>
  <text class="bs" x="308" y="259.0">top-k by cosine</text>
  <rect class="bx" x="384" y="224" width="104" height="44" rx="5"/>
  <text class="bt" x="436" y="245.0">prompt</text>
  <text class="bs" x="436" y="259.0">context + question</text>
  <rect class="bx" x="512" y="224" width="112" height="44" rx="5"/>
  <text class="bt" x="568" y="245.0">answer</text>
  <text class="bs" x="568" y="259.0">grounded + cited</text>
  <rect class="sw" x="16" y="224" width="104" height="44" rx="5"><animate attributeName="x" dur="7.5s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1667;0.3333;0.5000;0.6667" values="16;144;256;384;512"/><animate attributeName="width" dur="7.5s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1667;0.3333;0.5000;0.6667" values="104;88;104;104;112"/></rect>
  <rect class="sw" x="424" y="36" width="150" height="54" rx="5" opacity="0"><animate attributeName="opacity" dur="7.5s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.333;0.5" values="0;1;0"/></rect>
  <text class="lane" x="36" y="24">offline · indexing, runs once per document</text>
  <text class="lane on" x="16" y="292">online · retrieval + generation, runs on every question</text>
</svg>
<figcaption><b>Figure 1 -</b> The two halves of a RAG system. The top lane is paid once : documents are split, embedded and stored in an index. The bottom lane runs on every question, and touches only $k$ passages. The green route is the one link between them — the query embedding is compared against the index built above.</figcaption>
</center></figure>

Notice the asymmetry in the figure : the top lane runs **once** (or whenever documents change), the bottom lane runs on **every single question**. That is what makes the design practical — the expensive work of reading the whole corpus is amortised into an index, and a query only touches $k$ passages.

##1. Turning meaning into geometry : embeddings
For the librarian to work, "relevant to this question" must become something a computer can compute. Keyword matching is not enough : a user asking *"how do I reset my password ?"* should be handed a page titled *"Recovering account access"*, which shares not a single content word with the question.

The tool for that is the **embedding** : a function $\phi$ mapping a piece of text to a vector in $\mathbb{R}^d$ (typically $d$ between $384$ and $3072$), trained so that texts with similar meaning land close together. Modern embedders are Transformer encoders trained with a contrastive objective — pairs that mean the same thing are pulled together, unrelated pairs pushed apart {% cite Reimers2019 %}. Training them specifically on question–passage pairs, rather than on generic sentence similarity, is what made dense retrieval competitive with keyword search in the first place {% cite Karpukhin2020 %}. The result is a space where *direction encodes meaning*.

Which is why the similarity measure is the **cosine** of the angle between two vectors, not their distance :

<div class="definition">
	The <b>cosine similarity</b> between two non-zero vectors $u, v \in \mathbb{R}^d$ is
	$$ \operatorname{sim}(u,v) \;=\; \cos\theta \;=\; \frac{u \cdot v}{\lVert u \rVert\, \lVert v \rVert} \;\in\; [-1, 1], $$
	equal to $1$ when they point the same way, $0$ when orthogonal, $-1$ when opposite.
</div>

Why the angle and not the raw distance ? Because **length is noise here**. A long chunk that repeats a term produces a longer vector than a short chunk that says the same thing once ; ranking by dot product alone would systematically prefer the verbose one. Dividing by the norms throws that away and keeps only the direction — the meaning.

<figure><center>
<svg viewBox="0 0 640 312" width="100%" style="max-width:625px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .ax{ stroke:#d8d8d8; stroke-width:1.2; fill:none; }
    .dv{ stroke:#8fa5c8; stroke-width:1.6; fill:none; }
    .qv{ stroke:#b5651d; stroke-width:2.6; fill:none; }
    .dl{ font-size:11.5px; fill:#4a6da7; text-anchor:middle;
         paint-order:stroke; stroke:#fff; stroke-width:3.5px; stroke-linejoin:round; }
    .ql{ font-size:12px; fill:#b5651d; text-anchor:middle; font-weight:bold;
         paint-order:stroke; stroke:#fff; stroke-width:3.5px; stroke-linejoin:round; }
    .arc{ stroke:#b5651d; stroke-width:1.3; fill:none; opacity:.8; }
    .th{ font-size:13px; fill:#b5651d; text-anchor:middle; font-style:italic; }
    .bar{ fill:#8fa5c8; }
    .top{ fill:#e0a878; }
    .rl{ font-size:12px; fill:#4a6da7; text-anchor:end; }
    .bv{ font-size:11px; fill:#5a5a5a; text-anchor:start; }
    .hd2{ font-size:12px; fill:#7a7a7a; text-anchor:middle; font-style:italic; }
    .mk2{ fill:none; stroke:#b5651d; stroke-width:2; }
    .cap2{ font-size:11px; fill:#7a7a7a; text-anchor:middle; }
  </style>
  <defs>
    <marker id="am" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#4a6da7"/></marker>
    <marker id="ao" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#b5651d"/></marker>
    <marker id="ag" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#5aa06a"/></marker>
  </defs>
  <path class="ax" d="M58,268 L306,268 M58,268 L58,44"/>
  <path class="dv" d="M58,268 L167.6,127.7" marker-end="url(#am)"/>
  <path class="dv" d="M58,268 L226.9,170.5" marker-end="url(#am)"/>
  <path class="dv" d="M58,268 L119.8,115.0" marker-end="url(#am)"/>
  <path class="dv" d="M58,268 L209.6,235.8" marker-end="url(#am)"/>
  <path class="dv" d="M58,268 L70.6,123.6" marker-end="url(#am)"/>
  <path class="dv" d="M58,268 L182.8,261.5" marker-end="url(#am)"/>
  <path class="qv" d="M58,268 L199.4,126.6" marker-end="url(#ao)"/>
  <text class="dl" x="178.1" y="114.3">d1</text>
  <text class="dl" x="241.6" y="162.0">d2</text>
  <text class="dl" x="126.2" y="99.3">d3</text>
  <text class="dl" x="226.2" y="232.2">d4</text>
  <text class="dl" x="72.1" y="106.6">d5</text>
  <text class="dl" x="199.8" y="260.6">d6</text>
  <text class="ql" x="211.4" y="114.6">query</text>
  <path class="arc" d="M128.7,197.3 A100,100 0 0 0 95.5,175.3"/>
  <text class="th" x="122.6" y="170.4">θ</text>
  <text class="cap2" x="150" y="300">length is meaning-free : only the angle to the query counts</text>
  <text class="hd2" x="477" y="66">cosine with the query</text>
  <text class="rl" x="362" y="107">d1</text>
  <rect class="bar top" x="372" y="92" width="208.4" height="20" rx="3"/>
  <text class="bv" x="596" y="107">0.993</text>
  <text class="rl" x="362" y="139">d2</text>
  <rect class="bar top" x="372" y="124" width="202.8" height="20" rx="3"/>
  <text class="bv" x="596" y="139">0.966</text>
  <text class="rl" x="362" y="171">d3</text>
  <rect class="bar top" x="372" y="156" width="193.3" height="20" rx="3"/>
  <text class="bv" x="596" y="171">0.921</text>
  <text class="rl" x="362" y="203">d4</text>
  <rect class="bar" x="372" y="188" width="176.1" height="20" rx="3"/>
  <text class="bv" x="596" y="203">0.839</text>
  <text class="rl" x="362" y="235">d5</text>
  <rect class="bar" x="372" y="220" width="160.9" height="20" rx="3"/>
  <text class="bv" x="596" y="235">0.766</text>
  <text class="rl" x="362" y="267">d6</text>
  <rect class="bar" x="372" y="252" width="156.1" height="20" rx="3"/>
  <text class="bv" x="596" y="267">0.743</text>
  <rect class="mk2" x="367" y="92" width="220" height="20" rx="3"><animate attributeName="y" dur="6s" repeatCount="indefinite" calcMode="discrete" keyTimes="0;0.333;0.666" values="92;124;156"/></rect>
  <text class="cap2" x="477" y="300">the top three are what the retriever returns</text>
</svg>
<figcaption><b>Figure 2 -</b> Retrieval as geometry. Each document is a direction in embedding space ; the query is another, and $\theta$ (drawn here between the query and $d_3$) is the angle between them. Ranking by $\cos\theta$ ignores the vectors' lengths entirely, so a short passage and a long one that mean the same thing score alike. The bars on the right are the same six documents, sorted by that cosine.</figcaption>
</center></figure>

In practice every vector is normalised to unit length once, at index time. That is not just an optimisation, it makes two apparently different questions the same question :

<div class="theorem" text="cosine and distance agree on the unit sphere">
	If $\lVert u \rVert = \lVert v \rVert = 1$, then
	$$ \lVert u - v \rVert^{2} \;=\; 2\big(1 - \cos\theta\big). $$
	Consequently, ranking candidates by <i>decreasing</i> cosine similarity and ranking them by <i>increasing</i> Euclidean distance produce exactly the same order.
	<br/><br/>
	<i>Proof.&nbsp;&nbsp;</i> Expanding the squared norm,
	$$ \lVert u - v \rVert^{2} = (u-v)\cdot(u-v) = \lVert u \rVert^{2} + \lVert v \rVert^{2} - 2\,u \cdot v = 1 + 1 - 2\cos\theta. $$
	The right-hand side is a strictly decreasing function of $\cos\theta$, so the two orderings are reverses of one another and select the same top-$k$.
	<p align="right">$\square$</p>
</div>

This is a genuinely useful fact : it means a vector database that only knows how to find Euclidean nearest neighbours is a perfectly good cosine search engine, provided you normalise on the way in. It also means the dot product alone suffices, since $u \cdot v = \cos\theta$ for unit vectors — and a dot product against a whole index is one matrix multiplication.

##1. Chunking : the boring step that decides everything
You cannot embed a 400-page manual as one vector. A single vector has a fixed budget of meaning ; average an entire manual into it and you get a vector that means *"this is a manual"* and nothing else. So documents are cut into **chunks**, and each chunk is embedded separately.

This is where most RAG systems are silently won or lost, because chunking sets a hard ceiling on what retrieval can ever do :

- **Chunks too large** : the embedding is diluted (many topics averaged into one direction), and each retrieved chunk burns context on mostly irrelevant text.
- **Chunks too small** : the chunk no longer carries enough context to be interpretable. A chunk reading *"It must be renewed every 90 days."* is useless — the reader cannot tell what "it" is.

The standard mitigation is **overlap** : consecutive chunks share a tail, so a sentence near a boundary appears in both and keeps its neighbourhood. Overlap is not free, and the cost is easy to quantify. With chunk size $c$ and overlap $o < c$, consecutive chunks start every $c - o$ characters, so a document of length $L$ produces

$$ \begin{equation} n \;=\; \left\lceil \frac{L - o}{\,c - o\,} \right\rceil \quad\text{chunks, storing}\quad \frac{n\,c}{L} \;\approx\; \frac{c}{c - o} \quad\text{times the original text.} \label{eq:chunks} \end{equation} $$

*In plain words : a 50 % overlap ($o = c/2$) doubles the size of your index, and a 75 % overlap quadruples it.* That is the trade you are making — index size and query cost against the risk of cutting an answer in half.

<div class="note">
	The best chunking is usually not a fixed character count at all, but the document's own structure : split on headings, on Markdown sections, on function definitions, on table rows. A chunk that corresponds to a real semantic unit needs far less overlap, because its boundaries were never arbitrary in the first place. Fixed-size chunking is the fallback for unstructured text, not the default.
</div>

##1. Finding the neighbours fast
Now the search problem. Given the query vector $q$ and an index of $N$ chunk vectors, we want the $k$ largest cosines. Exact search computes all $N$ dot products :

$$ \begin{equation} \text{cost} \;=\; \mathcal{O}(N d) \quad\text{per query.} \label{eq:exact} \end{equation} $$

For $N = 10^{4}$ that is nothing — a single NumPy matrix multiply, microseconds, and you should absolutely do it. For $N = 10^{8}$ chunks and $d = 1024$ it is $10^{11}$ multiply-adds per question, which is not a search engine, it is a batch job.

So production systems give up exactness. **Approximate nearest neighbour** (ANN) search accepts a small probability of missing a true neighbour in exchange for orders of magnitude in speed — the very same bargain struck by [Bloom filters]({{ site.baseurl }}/blog/algorithmic/bloom-filters) (a small false-positive rate buys a tiny memory footprint) and [HyperLogLog]({{ site.baseurl }}/blog/algorithmic/hyperloglog) (a few percent of counting error buys kilobytes instead of gigabytes). Approximation is the standard currency for buying scale.

The dominant method is **HNSW**, *Hierarchical Navigable Small World* graphs {% cite Malkov2020 %}. The idea is a skip list in disguise :

1. Build a graph where each vector is a node linked to some of its nearest neighbours.
2. Stack several such graphs in layers. A node appears in layer $\ell$ with probability decaying geometrically, so the top layer holds a handful of nodes and the bottom layer holds all $N$.
3. To search, enter at the top layer and walk **greedily** — repeatedly step to the neighbour closest to the query — until no neighbour improves. Then drop one layer and repeat, starting from where you landed.

<figure><center>
<svg viewBox="0 0 640 340" width="100%" style="max-width:625px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .eg{ stroke:#c2ccdc; stroke-width:1.6; fill:none; }
    .drop{ stroke:#c2ccdc; stroke-width:1.3; fill:none; stroke-dasharray:3 4; }
    .hi{ stroke:#b5651d; stroke-width:2.6; }
    .nd{ stroke:#4a6da7; stroke-width:1.5; }
    .ent{ fill:none; stroke:#5aa06a; stroke-width:1.8; stroke-dasharray:4 3; }
    .tag{ font-size:11px; fill:#5aa06a; text-anchor:middle; }
    .hit{ fill:#b5651d; }
    .ly{ font-size:12px; fill:#4a6da7; text-anchor:start; }
    .lyr{ font-size:11px; fill:#7a7a7a; text-anchor:end; font-style:italic; }
  </style>
  <path class="drop" d="M145.0,74.0 L145.0,168.0"/>
  <path class="drop hi" d="M365.0,74.0 L365.0,168.0"/>
  <path class="drop" d="M520.0,74.0 L520.0,168.0"/>
  <path class="drop" d="M90.0,188.0 L90.0,282.0"/>
  <path class="drop" d="M145.0,188.0 L145.0,282.0"/>
  <path class="drop" d="M255.0,188.0 L255.0,282.0"/>
  <path class="drop" d="M365.0,188.0 L365.0,282.0"/>
  <path class="drop hi" d="M470.0,188.0 L470.0,282.0"/>
  <path class="drop" d="M520.0,188.0 L520.0,282.0"/>
  <path class="eg hi" d="M155.0,64.0 L355.0,64.0"/>
  <path class="eg" d="M375.0,64.0 L510.0,64.0"/>
  <path class="eg" d="M100.0,178.0 L135.0,178.0"/>
  <path class="eg" d="M155.0,178.0 L245.0,178.0"/>
  <path class="eg" d="M265.0,178.0 L355.0,178.0"/>
  <path class="eg hi" d="M375.0,178.0 L460.0,178.0"/>
  <path class="eg" d="M480.0,178.0 L510.0,178.0"/>
  <path class="eg" d="M50.0,292.0 L80.0,292.0"/>
  <path class="eg" d="M100.0,292.0 L135.0,292.0"/>
  <path class="eg" d="M155.0,292.0 L190.0,292.0"/>
  <path class="eg" d="M210.0,292.0 L245.0,292.0"/>
  <path class="eg" d="M265.0,292.0 L300.0,292.0"/>
  <path class="eg" d="M320.0,292.0 L355.0,292.0"/>
  <path class="eg" d="M375.0,292.0 L410.0,292.0"/>
  <path class="eg" d="M430.0,292.0 L460.0,292.0"/>
  <path class="eg hi" d="M480.0,292.0 L510.0,292.0"/>
  <path class="eg" d="M530.0,292.0 L560.0,292.0"/>
  <path class="eg" d="M580.0,292.0 L605.0,292.0"/>
  <circle class="nd" cx="145" cy="64" r="9" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1429;0.2857;0.4286;0.5714;0.7143" values="#e8b48a;#e8b48a;#e8b48a;#e8b48a;#e8b48a;#e8b48a"/></circle>
  <circle class="nd" cx="365" cy="64" r="9" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1429;0.2857;0.4286;0.5714;0.7143" values="#f0f2f7;#e8b48a;#e8b48a;#e8b48a;#e8b48a;#e8b48a"/></circle>
  <circle class="nd" cx="520" cy="64" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="90" cy="178" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="145" cy="178" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="255" cy="178" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="365" cy="178" r="9" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1429;0.2857;0.4286;0.5714;0.7143" values="#f0f2f7;#f0f2f7;#e8b48a;#e8b48a;#e8b48a;#e8b48a"/></circle>
  <circle class="nd" cx="470" cy="178" r="9" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1429;0.2857;0.4286;0.5714;0.7143" values="#f0f2f7;#f0f2f7;#f0f2f7;#e8b48a;#e8b48a;#e8b48a"/></circle>
  <circle class="nd" cx="520" cy="178" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="40" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="90" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="145" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="200" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="255" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="310" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="365" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="420" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="470" cy="292" r="9" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1429;0.2857;0.4286;0.5714;0.7143" values="#f0f2f7;#f0f2f7;#f0f2f7;#f0f2f7;#e8b48a;#e8b48a"/></circle>
  <circle class="nd" cx="520" cy="292" r="9" fill="#f0f2f7"><animate attributeName="fill" dur="8s" repeatCount="indefinite" calcMode="discrete" keyTimes="0.0000;0.1429;0.2857;0.4286;0.5714;0.7143" values="#f0f2f7;#f0f2f7;#f0f2f7;#f0f2f7;#f0f2f7;#e8b48a"/></circle>
  <circle class="nd" cx="570" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="nd" cx="615" cy="292" r="9" fill="#f0f2f7"/>
  <circle class="ent" cx="145" cy="64" r="15"/>
  <text class="tag" x="145" y="38">entry</text>
  <text class="tag hit" x="520" y="322">target</text>
  <text class="ly" x="18" y="40">layer 2</text>
  <text class="lyr" x="628" y="40">3 nodes · very long hops</text>
  <text class="ly" x="18" y="154">layer 1</text>
  <text class="lyr" x="628" y="154">6 nodes · medium hops</text>
  <text class="ly" x="18" y="268">layer 0</text>
  <text class="lyr" x="628" y="268">every node · short hops</text>
</svg>
<figcaption><b>Figure 3 -</b> An HNSW index. A node appears in a layer with geometrically decaying probability, so the top layer is sparse and its links span the whole space. The search enters at the top, walks greedily to the closest node it can reach, drops one layer (dashed link, same node) and repeats. The orange trail is one query : two hops at the top replace hundreds at the bottom.</figcaption>
</center></figure>

*In plain words : the top layers are the motorway network, with a few nodes and very long hops that cross the whole space in a couple of moves. The bottom layer is the local street map. You take the motorway to get roughly there, then exit and navigate street by street.* Each layer contributes a constant expected number of hops, and there are $\mathcal{O}(\log N)$ layers, so a query costs

$$ \begin{equation} \mathcal{O}(d \log N) \quad\text{instead of}\quad \mathcal{O}(N d). \label{eq:hnsw} \end{equation} $$

At $N = 10^{8}$ that is the difference between a hundred million comparisons and a few hundred. The price is **recall below 1** : the greedy walk can get stuck in a local minimum and miss a true neighbour. Every ANN index exposes a knob (`efSearch` in HNSW, `nprobe` in IVF-style indexes {% cite Johnson2021 %}) that trades latency for recall, and that knob is one of the few in a RAG system with an honest, measurable meaning.

<div class="note">
	Do not reach for an ANN index by reflex. Below roughly $10^{5}$ chunks, brute-force cosine on a normalised matrix is faster than most ANN libraries once you count index build time, it is exact, and it has no tuning knobs to get wrong. Reach for HNSW when $\eqref{eq:exact}$ actually hurts — not before.
</div>

##1. Where dense retrieval fails : lexical search and hybrid
Embeddings are excellent at *meaning* and surprisingly bad at *strings*. Ask for error code `ORA-01555` or part number `MX-4471-B`, and the embedder — which never saw that token during training — maps it to a vague direction near "technical identifier". Semantically similar, lexically useless : you will get the page about a *different* error code.

The classical answer is still the right one. **BM25** ranks documents by weighted term overlap {% cite Robertson2009 %} :

$$ \begin{equation} \operatorname{BM25}(q,d) \;=\; \sum_{t \in q} \operatorname{IDF}(t)\; \cdot\; \frac{f(t,d)\,(k_1 + 1)}{f(t,d) + k_1\Big(1 - b + b\,\dfrac{\lvert d \rvert}{\text{avgdl}}\Big)}, \label{eq:bm25} \end{equation} $$

where $f(t,d)$ is how often term $t$ occurs in document $d$, $\lvert d \rvert$ is the document length, $\text{avgdl}$ the mean length, and $k_1 \approx 1.5$, $b \approx 0.75$ are the usual constants. Two ideas are worth extracting from that formula, because both are good engineering :

- **Saturation.** As $f(t,d) \to \infty$ the fraction tends to $k_1 + 1$, a finite ceiling. A document mentioning a term twenty times is *not* twenty times more relevant than one mentioning it once — the tenth occurrence tells you almost nothing new. $k_1$ controls how fast the credit saturates.
- **Length normalisation.** The $b \frac{\lvert d \rvert}{\text{avgdl}}$ term penalises long documents, which would otherwise win purely by containing more words. It is the same instinct as dividing by $\lVert v \rVert$ in the cosine.

And $\operatorname{IDF}(t) = \ln\!\left(\frac{N - n_t + 0.5}{n_t + 0.5} + 1\right)$, with $n_t$ the number of documents containing $t$, makes rare terms count for much more than common ones — which is exactly why BM25 nails the part number that the embedder smeared away.

So run **both** retrievers and merge. The catch is that a BM25 score of $14.2$ and a cosine of $0.83$ live on incomparable scales, and normalising them is a fiddly, corpus-dependent mess. The robust trick is to throw the scores away and keep only the **ranks** — Reciprocal Rank Fusion {% cite Cormack2009 %} :

$$ \begin{equation} \operatorname{RRF}(d) \;=\; \sum_{r \in \text{rankers}} \frac{1}{\kappa + \operatorname{rank}_r(d)}, \qquad \kappa \approx 60. \label{eq:rrf} \end{equation} $$

*In plain words : each ranker votes, a first place is worth more than a second, and nobody has to agree on what a "score" means.* The constant $\kappa$ flattens the top of the curve so that the difference between rank 1 and rank 2 does not overwhelm the fact that a document was ranked well by *both* systems.

```python
def rrf(rankings, kappa=60):
    """Fuse ranked ID lists. Scores are never compared."""
    fused = {}
    for ranking in rankings:
        for r, doc in enumerate(ranking, start=1):
            fused[doc] = fused.get(doc, 0) + 1 / (kappa + r)
    return sorted(fused, key=fused.get, reverse=True)
```

Hybrid retrieval is, in my experience, the single highest-return change in a mediocre RAG system — well ahead of swapping the generator for a bigger one.

##1. Reranking : cheap recall, then expensive precision
There is a structural reason retrieval is not very precise, and understanding it explains the standard architecture.

The retriever uses a **bi-encoder** : the question and the chunk are encoded *separately*, into $\phi(q)$ and $\phi(c)$, and compared by a cosine. That separation is what makes indexing possible — every $\phi(c)$ is computed once, offline, and a query only needs one new embedding. But it also means the encoder must summarise a chunk into a single vector **without knowing what will be asked**. Information is necessarily lost.

A **cross-encoder** does the opposite : it feeds the pair $(q, c)$ through a Transformer *together*, so every token of the question can attend to every token of the chunk before a single relevance score comes out {% cite Nogueira2019 %}. Far more accurate — and unusable as a search index, because nothing can be precomputed : ranking $N$ chunks needs $N$ forward passes *per query*.

<figure><center>
<svg viewBox="0 0 640 234" width="100%" style="max-width:625px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .bx{ fill:#f6f7fb; stroke:#4a6da7; stroke-width:1.5; }
    .cs{ fill:#f3faf4; stroke:#5aa06a; }
    .cx{ fill:#fbeede; stroke:#b5651d; stroke-width:1.8; }
    .bt{ font-size:12.5px; fill:#2b2b2b; text-anchor:middle; }
    .bs{ font-size:9.5px; fill:#7a7a7a; text-anchor:middle; }
    .ln{ stroke:#4a6da7; stroke-width:1.7; fill:none; }
    .lo{ stroke:#b5651d; stroke-width:1.7; fill:none; }
    .div{ stroke:#dcdcdc; stroke-width:1.2; }
    .vec{ fill:#eef1f8; stroke:#4a6da7; stroke-width:1.5; }
    .sc{ fill:#fbeede; stroke:#b5651d; stroke-width:1.8; }
    .vt{ font-size:11.5px; fill:#2b2b2b; text-anchor:middle; font-style:italic; }
    .ttl{ font-size:12.5px; fill:#4a6da7; text-anchor:middle; font-style:italic; }
    .or{ fill:#b5651d; }
    .nt{ font-size:11px; fill:#5a5a5a; text-anchor:middle; }
    .st{ fill:#7a7a7a; }
  </style>
  <defs>
    <marker id="am" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#4a6da7"/></marker>
    <marker id="ao" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#b5651d"/></marker>
    <marker id="ag" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5.5" markerHeight="5.5" orient="auto-start-reverse">
      <path d="M0,1 L9,5 L0,9 z" fill="#5aa06a"/></marker>
  </defs>
  <path class="div" d="M318,32 L318,224"/>
  <text class="ttl" x="160" y="24">bi-encoder — builds the index</text>
  <text class="ttl or" x="480" y="24">cross-encoder — reranks the shortlist</text>
  <rect class="bx" x="20" y="56" width="68" height="30" rx="5"/>
  <text class="bt" x="54" y="75.0">query</text>
  <rect class="bx" x="20" y="140" width="68" height="30" rx="5"/>
  <text class="bt" x="54" y="159.0">chunk</text>
  <rect class="bx" x="108" y="56" width="74" height="30" rx="5"/>
  <text class="bt" x="145" y="75.0">encoder</text>
  <rect class="bx" x="108" y="140" width="74" height="30" rx="5"/>
  <text class="bt" x="145" y="159.0">encoder</text>
  <path class="ln" d="M88.0,71.0 L101.0,71.0" marker-end="url(#am)"/>
  <path class="ln" d="M88.0,155.0 L101.0,155.0" marker-end="url(#am)"/>
  <path class="ln" d="M182.0,71.0 L195.0,71.0" marker-end="url(#am)"/>
  <path class="ln" d="M182.0,155.0 L195.0,155.0" marker-end="url(#am)"/>
  <circle class="vec" cx="208" cy="71" r="13"/><text class="vt" x="208" y="76">q</text>
  <circle class="vec" cx="208" cy="155" r="13"/><text class="vt" x="208" y="160">c</text>
  <rect class="bx cs" x="240" y="98" width="66" height="30" rx="5"/>
  <text class="bt" x="273" y="117.0">cos</text>
  <path class="ln" d="M221,79 L237,100" marker-end="url(#am)"/>
  <path class="ln" d="M221,147 L237,126" marker-end="url(#am)"/>
  <text class="nt" x="160" y="204">the chunk tower runs offline, once per chunk :</text>
  <text class="nt st" x="160" y="219">a query costs one encode + one index lookup</text>
  <rect class="bx" x="340" y="56" width="68" height="30" rx="5"/>
  <text class="bt" x="374" y="75.0">query</text>
  <rect class="bx" x="340" y="140" width="68" height="30" rx="5"/>
  <text class="bt" x="374" y="159.0">chunk</text>
  <rect class="bx cx" x="438" y="84" width="90" height="58" rx="5"/>
  <text class="bt" x="483" y="112.0">encoder</text>
  <text class="bs" x="483" y="126.0">both at once</text>
  <path class="lo" d="M408,71 L434,98" marker-end="url(#ao)"/>
  <path class="lo" d="M408,155 L434,128" marker-end="url(#ao)"/>
  <path class="lo" d="M528,113 L545,113" marker-end="url(#ao)"/>
  <circle class="sc" cx="570" cy="113" r="19"/><text class="vt" x="570" y="117">score</text>
  <text class="nt or" x="480" y="204">nothing can be precomputed :</text>
  <text class="nt or st" x="480" y="219">one full pass per (query, chunk) pair</text>
</svg>
<figcaption><b>Figure 4 -</b> The two ways to score a (query, chunk) pair. The bi-encoder keeps them apart, which loses information but lets every chunk vector be computed once and stored — that is what an index <i>is</i>. The cross-encoder lets the two texts attend to each other, which is far more accurate and impossible to precompute. Retrieval uses the first, reranking the second.</figcaption>
</center></figure>

Hence the two-stage design that every serious system converges on :

$$ \begin{equation} \underbrace{N = 10^{6}\;\text{chunks}}_{\text{corpus}} \;\xrightarrow[\ \mathcal{O}(d\log N)\ ]{\text{bi-encoder + ANN}}\; \underbrace{50\;\text{candidates}}_{\text{high recall}} \;\xrightarrow[\ 50\ \text{forward passes}\ ]{\text{cross-encoder}}\; \underbrace{5\;\text{passages}}_{\text{high precision}} \label{eq:twostage} \end{equation} $$

*In plain words : cast a wide, cheap net to make sure the answer is somewhere in the catch, then pay for a careful reading of fifty candidates to pick the best five.* The first stage is optimised for **recall** (do not lose the right chunk), the second for **precision** (put it first). Reranking fifty candidates is a fixed, affordable cost that does not grow with the corpus.

##1. The generation half
The retrieved passages now go into the prompt. This half is shorter, but two things matter.

**The instruction must be grounding, not decoration.** The generator's default behaviour is to answer from parametric memory ; it has to be told, explicitly, to prefer the context, to cite, and — critically — that *refusing is an acceptable answer*. Without that last clause the model will always produce something.

**Structure the context.** Number the passages and ask for the numbers back. Citations are not a nicety : they are what makes the output *checkable*, and a claim carrying a chunk number can be verified mechanically against the chunk.

```python
import numpy as np
import anthropic

client = anthropic.Anthropic()

SYSTEM = (
    "Answer using only the numbered context passages. "
    "Cite the passage number for every claim, like [3]. "
    "If the context does not contain the answer, say so "
    "plainly instead of guessing."
)

def top_k(query_vec, index, k=5):
    """index : (N, d) matrix of unit-norm chunk vectors."""
    scores = index @ query_vec        # cosine, rows are unit
    best = np.argpartition(-scores, k)[:k]
    return best[np.argsort(-scores[best])]

def answer(question, chunks, hits):
    context = "\n\n".join(f"[{i}] {chunks[i]}" for i in hits)
    prompt = f"<context>\n{context}\n</context>\n\n{question}"
    response = client.messages.create(
        model="claude-opus-5",
        max_tokens=16000,
        system=SYSTEM,
        messages=[{"role": "user", "content": prompt}],
    )
    return "".join(
        b.text for b in response.content if b.type == "text"
    )
```

That is a complete, working RAG loop in about twenty lines. Everything else in this post — chunking strategy, hybrid search, ANN indexes, reranking — is about making the `hits` on line three of `answer` contain the right passages. Which brings us to the only thing that tells you whether they do.

##1. Measuring it, and the ceiling you cannot cross
RAG systems fail in two completely different ways, and lumping them together is why so many are tuned by superstition. **Always evaluate the two stages separately.**

**Retrieval metrics** need a set of questions with known relevant chunks.

$$ \begin{equation} \operatorname{Recall}@k = \frac{\lvert R(q) \cap \text{relevant}(q) \rvert}{\lvert \text{relevant}(q) \rvert}, \qquad \operatorname{MRR} = \frac{1}{\lvert Q \rvert}\sum_{q \in Q} \frac{1}{\operatorname{rank}_q}, \label{eq:recall} \end{equation} $$

where $\operatorname{rank}_q$ is the position of the first relevant chunk. Recall@$k$ asks *"did we get it at all ?"*, MRR asks *"how near the top ?"*. When relevance has degrees, nDCG {% cite Jarvelin2002 %} weights each hit by its usefulness and discounts it by its position :

$$ \begin{equation} \operatorname{DCG}@k = \sum_{i=1}^{k} \frac{2^{\text{rel}_i} - 1}{\log_2(i+1)}, \qquad \operatorname{nDCG}@k = \frac{\operatorname{DCG}@k}{\operatorname{IDCG}@k}. \label{eq:ndcg} \end{equation} $$

**Generation metrics** are about faithfulness : is every claim in the answer supported by a retrieved passage, and does the answer address the question ? Both are usually scored by a second model acting as a judge, against the passages actually shown.

Now the result that should govern where you spend your time :

<div class="theorem" text="the retrieval ceiling">
	Let $A$ be the event that the system answers a question correctly and with grounding, and let $\mathcal{E}$ be the event that at least one passage containing the necessary evidence is in $R(q)$. Since the generator sees nothing but $q$ and $R(q)$, an answer that is <i>grounded</i> in evidence is impossible when no such evidence was retrieved, so $A \subseteq \mathcal{E}$ and therefore
	$$ \mathbb{P}(A) \;\le\; \mathbb{P}(\mathcal{E}) \;=\; \operatorname{Recall}@k. $$
</div>

*In plain words : if the right chunk is only retrieved 70 % of the time, your system cannot be right more than 70 % of the time — no matter which model you plug in, how you word the prompt, or how much you pay per token.* The generator can only ever lose accuracy relative to this bound, never gain it.[^1]

This gives a diagnostic that costs nothing and settles most arguments. When an answer is wrong, look at what was retrieved **before** touching anything else :

- The right chunk was **not** retrieved → the fault is in chunking, embedding, or search. Changing the prompt is theatre.
- The right chunk **was** retrieved and the answer is still wrong → now it is a generation problem : prompt, context ordering, or model.

##1. Where RAG breaks
RAG is a retrieval system with a language model attached, and it inherits every limitation of retrieval.

**It cannot aggregate.** *"How many of our customers are on the enterprise plan ?"* requires scanning the whole corpus ; retrieval returns $k$ passages. The system will confidently count the ones it happened to see and report a number that is not merely wrong but *plausibly* wrong. RAG is not a database, and questions of the form *count / sum / compare all* need a different tool — usually generated SQL, or a [MapReduce]({{ site.baseurl }}/blog/algorithmic/mapreduce)-style pass over the full corpus.

**It cannot chain.** *"Who manages the person who wrote the incident report on the payments outage ?"* needs two hops : find the report's author, then look up their manager. One retrieval round keyed on the original question will rarely surface both facts, because the second query cannot be formed until the first is answered. Multi-hop demands an agentic loop that retrieves, reads, and retrieves again.

**Long context is not free attention.** Stuffing fifty passages in "just in case" degrades accuracy : models recover information near the beginning and the end of a long context far more reliably than in the middle, the *lost in the middle* effect {% cite Liu2024 %}. More context is not more knowledge, and the reranker's job is precisely to keep the number of passages small and their order meaningful.

**The index goes stale silently.** Embeddings are computed once. Change a document and the index still serves the old vector, with no error and no warning — the system confidently answers from a version of the truth that no longer exists. Re-indexing is an operational obligation, not an optimisation.

**Changing the embedding model invalidates everything.** Vectors from two different embedders are not comparable — they are coordinates in unrelated spaces. Upgrading the embedder means re-embedding the entire corpus, which for a large index is the most expensive routine operation in the whole system.

##1. Conclusion
Retrieval-Augmented Generation is best understood not as a clever prompting trick but as **an information retrieval system that happens to end in a language model**. That framing tells you where the difficulty is, and it is not where most teams look : it is in cutting documents into units that mean something, in mapping meaning to geometry well enough that a cosine ranks correctly, in combining semantic and lexical search because neither alone is sufficient, and in spending a little compute on a careful second read.

The retrieval ceiling is the sentence to remember. Your system cannot be more correct than its retrieval is complete, so the first question about any wrong answer is never *"which model should we try ?"* but *"was the right passage even in the prompt ?"* Answer that honestly, with measurements, and the rest of the engineering follows. Answer it by intuition, and you will spend months tuning the half that was never broken.


References
----------
{% bibliography --cited %}

[^1]: The bound is an upper limit on *grounded* correctness. A model can of course answer correctly from its own parametric memory when retrieval fails — but then the citation is missing or fabricated, and you have no way to distinguish that lucky case from a hallucination. Counting it as a success measures the wrong thing.
