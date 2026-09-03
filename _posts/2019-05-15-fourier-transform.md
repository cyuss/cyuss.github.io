---
layout: post
title: The Fourier Transform, Why Sinusoids?
date: 2019-05-15 12:00
comments: true
external-url:
categories: Mathematics
tags: [analysis, signal_processing, calculus]
permalink: /blog/:categories/fourier-transform
---


> Shine white light through a prism and it fans out into a rainbow : the prism reveals that "white" was secretly a mixture of every colour. The **Fourier transform** is a prism for *signals*. It takes a sound, an image, a stock price, any wiggly function of time, and reveals the hidden mixture of **pure tones** it is made of : how much of each frequency is present. This one idea underpins audio compression, image formats, quantum mechanics and, as we will see, the way [Transformers]({{ site.baseurl }}/blog/machine-learning/transformers) encode the position of a word. But it raises a nagging question : of all possible building blocks, why should *sine waves* be the right ones ? This post answers that, keeping the intuition simple and the mathematics honest.

##1. The big idea : signals are recipes of sine waves
A **sine wave** $\sin(2\pi f t)$ is the purest oscillation there is : a single, unwavering frequency $f$. The claim at the heart of Fourier analysis is startling in its generality.

<div class="definition">
	<b>Fourier's claim.</b> Essentially any periodic signal can be written as a sum of sine and cosine waves whose frequencies are whole-number multiples of a base frequency, each with its own amplitude. The list of those amplitudes is the signal's <b>spectrum</b>, its "recipe".
</div>

*In plain words : just as a chord is several pure notes played together, any repeating waveform, however jagged, is really a stack of simple sine waves added up. Fourier analysis hands you the sheet music.*

##1. Why sinusoids, and not something else ?
There are two deep reasons sine waves are the privileged ingredients.

- **They are "orthogonal".** Over one period, the average product of two sine waves of *different* whole-number frequencies is exactly zero. They do not interfere. This means we can measure "how much of frequency $n$ is present" without the other frequencies contaminating the answer, each ingredient can be extracted cleanly.
- **They are the natural modes of physics.** Sinusoids are the shapes that pass through any stable, linear, time-invariant system (a vibrating string, an electrical circuit, an echoing room) unchanged except for a rescaling. Feed in a pure tone and you get out the same tone, louder or softer and shifted, never a *different* frequency. No other family of functions has this property.

*In plain words : sine waves are chosen not by taste but by mathematics, they are the only building blocks that stay out of each other's way and that nature itself prefers.*

##1. The Fourier series
For a signal $f(t)$ that repeats with period $T$ (base frequency $\omega = 2\pi/T$), the recipe is written

$$ \begin{equation} f(t) = a_0 + \sum_{n=1}^{\infty}\Big( a_n \cos(n\omega t) + b_n \sin(n\omega t) \Big). \label{eq:series} \end{equation} $$

The amplitudes are recovered by the orthogonality trick above : to find how much of frequency $n$ is present, **multiply** the signal by that pure wave and **average** over a period (the contamination from all other frequencies integrates to zero) :

$$ \begin{equation} a_n = \frac{2}{T}\int_{0}^{T} f(t)\cos(n\omega t)\,dt, \qquad b_n = \frac{2}{T}\int_{0}^{T} f(t)\sin(n\omega t)\,dt. \label{eq:coeffs} \end{equation} $$

*In plain words : each integral in $\eqref{eq:coeffs}$ asks "how much does the signal resemble this particular pure wave ?" , it is a similarity score, exactly the kind of projection a [dot product]({{ site.baseurl }}/blog/machine-learning/attention-mechanism) performs, but for functions.*

##1. Watching a square wave being built
Nothing makes $\eqref{eq:series}$ more convincing than watching it work. A **square wave**, all sharp corners and flat tops, seems the least "sine-like" shape imaginable [^1]. Yet it is just

$$ f(t) = \frac{4}{\pi}\left( \sin t + \frac{\sin 3t}{3} + \frac{\sin 5t}{5} + \frac{\sin 7t}{7} + \cdots \right), $$

odd harmonics only, each a little weaker than the last. The animation adds them one at a time : with a single sine it is a gentle wave, and as more harmonics join, the corners sharpen and the tops flatten toward the ideal square (grey).

<figure><center>
<svg viewBox="0 0 640 260" width="100%" style="max-width:600px" xmlns="http://www.w3.org/2000/svg" font-family="'PT Serif', Georgia, serif">
  <style>
    .axis{ stroke:#ddd; stroke-width:1; }
    .target{ fill:none; stroke:#c9cdd6; stroke-width:1.6; stroke-dasharray:5 5; }
    .sum{ fill:none; stroke:#4a6da7; stroke-width:2.6; }
    .cap{ font-size:12px; fill:#7a7a7a; text-anchor:middle; }
  </style>
  <line class="axis" x1="60" y1="130" x2="600" y2="130"/>
  <path class="target" d="M60,60 L192.9,60 L195,200 L327.9,200 L330,60 L462.9,60 L465,200 L597.9,200 L600,60"/>
  <path class="sum" d="M60.0,130.0">
    <animate attributeName="d" dur="6s" repeatCount="indefinite"
      keyTimes="0;0.22;0.44;0.66;0.88;1"
      values="M60.0,130.0 L62.1,125.7 L66.2,117.1 L74.5,100.4 L82.8,84.8 L91.2,70.9 L99.5,59.2 L107.8,50.1 L116.1,44.0 L124.4,41.1 L132.7,41.5 L141.0,45.2 L149.3,52.1 L157.6,61.9 L165.9,74.2 L174.2,88.6 L182.5,104.5 L190.8,121.4 L195.0,130.0 L203.3,147.1 L211.6,163.6 L219.9,178.8 L228.2,192.3 L236.5,203.3 L244.8,211.7 L253.2,217.0 L261.5,219.1 L269.8,217.9 L278.1,213.3 L286.4,205.7 L294.7,195.3 L303.0,182.4 L311.3,167.6 L319.6,151.3 L327.9,134.3 L330.0,130.0 L338.3,112.9 L346.6,96.4 L354.9,81.2 L363.2,67.7 L371.5,56.7 L379.8,48.3 L388.2,43.0 L396.5,40.9 L404.8,42.1 L413.1,46.7 L421.4,54.3 L429.7,64.7 L438.0,77.6 L446.3,92.4 L454.6,108.7 L462.9,125.7 L465.0,130.0 L473.3,147.1 L481.6,163.6 L489.9,178.8 L498.2,192.3 L506.5,203.3 L514.8,211.7 L523.2,217.0 L531.5,219.1 L539.8,217.9 L548.1,213.3 L556.4,205.7 L564.7,195.3 L573.0,182.4 L581.3,167.6 L589.6,151.3 L597.9,134.3 L600.0,130.0;M60.0,130.0 L64.2,112.9 L72.5,81.8 L80.8,59.1 L89.1,47.6 L97.4,46.8 L105.7,53.5 L114.0,62.7 L122.3,69.3 L130.6,70.1 L138.9,64.8 L147.2,55.8 L155.5,48.0 L163.8,46.4 L172.2,55.1 L180.5,75.2 L188.8,104.6 L195.0,130.0 L203.3,163.4 L211.6,190.8 L219.9,208.1 L228.2,214.0 L236.5,210.5 L244.8,201.9 L253.2,193.4 L261.5,189.5 L269.8,191.9 L278.1,199.5 L286.4,208.6 L294.7,213.9 L303.0,210.6 L311.3,196.2 L319.6,171.0 L327.9,138.6 L330.0,130.0 L338.3,96.6 L346.6,69.2 L354.9,51.9 L363.2,46.0 L371.5,49.5 L379.8,58.1 L388.2,66.6 L396.5,70.5 L404.8,68.1 L413.1,60.5 L421.4,51.4 L429.7,46.1 L438.0,49.4 L446.3,63.8 L454.6,89.0 L462.9,121.4 L465.0,130.0 L473.3,163.4 L481.6,190.8 L489.9,208.1 L498.2,214.0 L506.5,210.5 L514.8,201.9 L523.2,193.4 L531.5,189.5 L539.8,191.9 L548.1,199.5 L556.4,208.6 L564.7,213.9 L573.0,210.6 L581.3,196.2 L589.6,171.0 L597.9,138.6 L600.0,130.0;M60.0,130.0 L64.2,104.6 L72.5,64.1 L80.8,47.3 L89.1,51.8 L97.4,63.5 L105.7,68.2 L114.0,62.7 L122.3,54.6 L130.6,53.4 L138.9,60.5 L147.2,67.6 L155.5,65.7 L163.8,54.7 L172.2,46.8 L180.5,57.5 L188.8,92.8 L195.0,130.0 L203.3,178.1 L211.6,207.5 L219.9,212.4 L228.2,202.2 L236.5,192.8 L244.8,193.6 L253.2,201.7 L261.5,207.2 L269.8,203.7 L278.1,195.3 L286.4,191.9 L294.7,199.2 L303.0,210.6 L311.3,202.5 L319.6,187.7 L327.9,142.9 L330.0,130.0 L338.3,81.9 L346.6,52.5 L354.9,47.6 L363.2,57.8 L371.5,67.2 L379.8,66.4 L388.2,58.3 L396.5,52.8 L404.8,56.3 L413.1,64.7 L421.4,68.1 L429.7,60.8 L438.0,49.4 L446.3,49.1 L454.6,72.3 L462.9,117.1 L465.0,130.0 L473.3,178.1 L481.6,207.5 L489.9,212.4 L498.2,202.2 L506.5,192.8 L514.8,193.6 L523.2,201.7 L531.5,207.2 L539.8,203.7 L548.1,195.3 L556.4,191.9 L564.7,199.2 L573.0,210.6 L581.3,202.5 L589.6,187.7 L597.9,142.9 L600.0,130.0;M60.0,130.0 L64.2,82.0 L72.5,48.1 L80.8,66.2 L89.1,59.0 L97.4,57.2 L105.7,64.2 L114.0,56.9 L122.3,60.4 L130.6,62.4 L138.9,56.2 L147.2,63.0 L155.5,59.8 L163.8,56.2 L172.2,67.1 L180.5,52.1 L188.8,64.3 L195.0,130.0 L203.3,207.2 L211.6,202.3 L219.9,194.4 L228.2,205.1 L236.5,197.7 L244.8,199.0 L253.2,203.3 L261.5,196.5 L269.8,201.7 L278.1,201.3 L286.4,196.1 L294.7,204.7 L303.0,197.5 L311.3,197.1 L319.6,211.9 L327.9,155.4 L330.0,130.0 L338.3,52.8 L346.6,57.7 L354.9,65.6 L363.2,54.9 L371.5,62.3 L379.8,61.0 L388.2,56.7 L396.5,63.5 L404.8,58.3 L413.1,58.7 L421.4,63.9 L429.7,55.3 L438.0,62.5 L446.3,62.9 L454.6,47.7 L462.9,104.6 L465.0,130.0 L473.3,195.7 L481.6,202.3 L489.9,194.4 L498.2,205.1 L506.5,197.7 L514.8,199.0 L523.2,203.3 L531.5,196.5 L539.8,201.7 L548.1,201.3 L556.4,196.1 L564.7,204.7 L573.0,197.5 L581.3,197.1 L589.6,211.9 L597.9,155.4 L600.0,130.0;M60.0,130.0 L64.2,82.0 L72.5,48.1 L80.8,66.2 L89.1,59.0 L97.4,57.2 L105.7,64.2 L114.0,56.9 L122.3,60.4 L130.6,62.4 L138.9,56.2 L147.2,63.0 L155.5,59.8 L163.8,56.2 L172.2,67.1 L180.5,52.1 L188.8,64.3 L195.0,130.0 L203.3,207.2 L211.6,202.3 L219.9,194.4 L228.2,205.1 L236.5,197.7 L244.8,199.0 L253.2,203.3 L261.5,196.5 L269.8,201.7 L278.1,201.3 L286.4,196.1 L294.7,204.7 L303.0,197.5 L311.3,197.1 L319.6,211.9 L327.9,155.4 L330.0,130.0 L338.3,52.8 L346.6,57.7 L354.9,65.6 L363.2,54.9 L371.5,62.3 L379.8,61.0 L388.2,56.7 L396.5,63.5 L404.8,58.3 L413.1,58.7 L421.4,63.9 L429.7,55.3 L438.0,62.5 L446.3,62.9 L454.6,47.7 L462.9,104.6 L465.0,130.0 L473.3,195.7 L481.6,202.3 L489.9,194.4 L498.2,205.1 L506.5,197.7 L514.8,199.0 L523.2,203.3 L531.5,196.5 L539.8,201.7 L548.1,201.3 L556.4,196.1 L564.7,204.7 L573.0,197.5 L581.3,197.1 L589.6,211.9 L597.9,155.4 L600.0,130.0"/>
  </path>
  <text class="cap" x="330" y="240">adding sine harmonics one by one (1 → 2 → 3 → 6 terms) ; grey dashed = ideal square wave</text>
</svg>
<figcaption><b>Figure 1 -</b> A square wave reconstructed from its Fourier series. Each new odd harmonic sharpens the corners. Infinitely many are needed for a perfect square, the little overshoot at the jumps never quite disappears (the <i>Gibbs phenomenon</i>).</figcaption>
</center></figure>

##1. From series to transform
The Fourier *series* handles periodic signals. For a signal that does **not** repeat, we let the period $T \to \infty$, the discrete list of harmonics blurs into a continuous range of frequencies, and the sum becomes an integral. Using Euler's identity $e^{i\theta} = \cos\theta + i\sin\theta$ to package sine and cosine together, we arrive at the **Fourier transform** :

$$ \begin{equation} \hat{f}(\xi) = \int_{-\infty}^{\infty} f(t)\, e^{-2\pi i \xi t}\, dt. \label{eq:ft} \end{equation} $$

*In plain words : $\hat{f}(\xi)$ is still just "how much of frequency $\xi$ is in the signal", the same similarity score as $\eqref{eq:coeffs}$, now measured for every frequency on a continuum.* The transform is invertible : $f$ and $\hat f$ are two views of the same object, one in the **time domain**, one in the **frequency domain**. A sharp spike in $\hat f$ means a strong pure tone ; a spread-out $\hat f$ means a rich mixture.

##1. Why it matters
Once a signal is in the frequency domain, things that were hard become easy. Removing hiss from audio is just deleting high-frequency components ; JPEG and MP3 compression throw away the frequencies your eyes and ears barely notice ; solving certain differential equations turns into simple algebra. The workhorse that makes this practical is the **Fast Fourier Transform** (FFT) {% cite Cooley1965 %}, which computes $\eqref{eq:ft}$ for $N$ samples in $N\log N$ operations instead of $N^2$, one of the most consequential algorithms ever written.

And there is a charming connection back to deep learning. The [Transformer]({{ site.baseurl }}/blog/machine-learning/transformers) tags each word position with a vector of sines and cosines *of many different frequencies*, exactly the ingredients of a Fourier series. Fast-oscillating components pin down fine position, slow ones the coarse position, so the whole vector is a Fourier-style "fingerprint" of where a word sits, the same principle that lets $\eqref{eq:coeffs}$ distinguish one frequency from another.

##1. Conclusion
The Fourier transform is a change of perspective : stop describing a signal by *what it does at each moment*, and start describing it by *which frequencies it is made of*. Sine waves earn their central role honestly, they are the only ingredients that stay out of each other's way and that linear physics leaves intact, so the recipe they define is unique and meaningful. From a prism splitting light to a Transformer locating a word, the same idea keeps reappearing : complicated things are often simple mixtures of pure oscillations, if only you look at them through the right prism.


References
----------
{% bibliography --cited %}

[^1]: Joseph Fourier introduced these series in 1822 while studying heat flow, to considerable scepticism, his contemporaries doubted that sums of smooth sines could represent functions with sharp corners. The Gibbs overshoot in Figure 1 is a trace of exactly that tension.
