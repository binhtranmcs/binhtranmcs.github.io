---
layout: post
title: ICPC 2022 Regional H, Hardest Problem
cover-img: /assets/img/fft-cover.png
thumbnail-img: /assets/img/fft-cover.png
share-img: /assets/img/fft-cover.png
tags: [ICPC, ACM, competitive programming, algorithms]
author: Binh Tran
mathjax: true
---

My most recent competitive programming event was the 2022 ICPC Asia Ho Chi Minh City Regional Contest, where the top four university teams qualified for the World Final. Our team finished in 18th place, making us the fifth highest, having solved one problem fewer than the fourth-ranked team. At the start of the contest, I was drawn to [Problem H](https://oj.vnoi.info/problem/icpc22_regional_h#comments), a math question that was arguably the most challenging of the competition. Confident in my math skills, I believed I could solve it, but I couldn't. After over an hour of struggle, I realized it was too difficult for me and decided to move on. I often think about how if I had set it aside sooner, we might have solved one more other problem and secured a place in the World Final. I took the problem statement home to try again but was still unsuccessful. Although this contest was three years ago, the problem statement remains on my desk. I recently found it while cleaning for the Lunar New Year, and it seems I still haven't moved on!

You can find the problem statement in the link above. TLDR:

Given two integers n and d. Define f(k) as the number of permutations of 1, 2, ..., n such that:
- The number of inversions of the permutation is k.
- When removing all elements with values that are strictly greater than d from the permutation, the remaining elements are sorted in increasing order.

Find f(k) modulo 998244353 for all k.

The solution was never revealed, but it was vaguely discussed in this [post](https://www.facebook.com/code.cung.rr/posts/pfbid02Xue7uxdVBTZqeEMTP7vBewiXQKsXzbK1GBMpX3DTuJRWZB8oRGejdZKHmNWfGyXMl?rdid=DraLgPH36bn3SC1M#). I will try to explain it in detail, so that I can understand :).

Below is a full explanation, powered by Claude.

## Step 1 — Model the permutation as insertions

Build the permutation by inserting values \\(1, 2, \ldots, n\\) one at a time, in increasing order. Each inserted value is the current maximum of everything placed so far, so it can go into any "gap" of the sequence built so far: if value \\(i\\) ends up with \\(t\\) elements after it, it creates exactly \\(t\\) new inversions.

This is the standard **inversion table / Lehmer code** idea — you can read more about it on [Wikipedia's inversion page](https://en.wikipedia.org/wiki/Inversion_(discrete_mathematics)).

Because values \\(1..d\\) must keep increasing relative order (removing everything greater than `d` must leave them sorted), each of them has **exactly one legal insertion position** — the very end of what's been placed — contributing 0 inversions. Values \\(d+1..n\\) are unconstrained.

## Step 2 — Per-insertion generating function

Inserting the current max into a sequence of length \\(i-1\\) gives \\(i\\) equally likely positions, contributing \\(0, 1, \ldots, i-1\\) extra inversions:

$$1+q+q^2+\cdots+q^{i-1}$$

This per-step factor is the standard building block for counting permutations by inversions, known as [Mahonian numbers](https://en.wikipedia.org/wiki/Permutation#Mahonian_numbers).

## Step 3 — Multiply independent choices

Since insertions are independent, the total generating function is the product:

$$F(q)=\prod_{i=d+1}^{n}\left(1+q+\cdots+q^{i-1}\right)$$

This follows from the basic rule that [generating functions of independent choices multiply](https://en.wikipedia.org/wiki/Generating_function#Ordinary_generating_function).

## Step 4 — Collapse each factor via geometric series

$$1+q+\cdots+q^{i-1}=\frac{1-q^{i}}{1-q}$$

Just the standard [geometric series formula](https://en.wikipedia.org/wiki/Geometric_series#Formula).

## Step 5 — Numerator / denominator form

Substituting Step 4 into Step 3:

$$F(q)=\frac{\prod_{i=d+1}^{n}(1-q^{i})}{(1-q)^{\,n-d}}$$

## Step 6 — Denominator: generalized binomial theorem

$$(1-q)^{-m}=\sum_{j\ge0}\binom{m-1+j}{j}q^{j}$$

Explicit coefficients, computed directly from precomputed factorials mod `p` — no series inversion needed. See the [binomial series](https://en.wikipedia.org/wiki/Binomial_series) identity and [cp-algorithms' binomial coefficients page](https://cp-algorithms.com/combinatorics/binomial-coefficients.html) for how to compute these mod a prime.

## Step 7 — Numerator: log of the product

$$\ln(1-q^{i})=-\sum_{j\ge1}\frac{q^{ij}}{j}$$

This is the [Mercator series](https://en.wikipedia.org/wiki/Mercator_series) for \\(\ln(1-x)\\). Summing over `i` and collecting by total degree \\(m = ij\\) gives a divisor-sum sieve:

$$L_m=-\sum_{\substack{i\mid m\\ d+1\le i\le n}}\frac{i}{m}$$

This sieve-over-divisors technique is the same style used for [divisor sum computations](https://cp-algorithms.com/algebra/divisors.html).

## Step 8 — Recover the numerator via power series exp

$$N(q)=\exp(L(q))\bmod q^{K+1}$$

computed via Newton's iteration, doubling precision each round:

$$N_t = N_{t-1}\cdot\bigl(1+L-\ln N_{t-1}\bigr) \pmod{q^{2^{t}}}$$

See [cp-algorithms' page on power series operations](https://cp-algorithms.com/algebra/polynomial.html) for the derivation of inverse/log/exp via Newton's method, and the [FFT/NTT page](https://cp-algorithms.com/algebra/fft.html) for the fast multiplication these all build on.

## Step 9 — Final convolution

$$f_k=\sum_{j=0}^{k}N_j\cdot D_{k-j}$$

One polynomial multiplication (NTT) of the numerator and denominator series, truncated to degree `K`.

## Summary

| Step | Formula | Concept | Reference |
| :--- | :--- | :--- | :--- |
| 1–3 | \\(F(q) = \prod(1+q+\cdots+q^{i-1})\\) | inversion generating functions | [Mahonian numbers](https://en.wikipedia.org/wiki/Permutation#Mahonian_numbers) |
| 4–5 | \\(F(q) = \prod(1-q^i) / (1-q)^{n-d}\\) | geometric series | [Geometric series](https://en.wikipedia.org/wiki/Geometric_series#Formula) |
| 6 | \\(D_j = \binom{n-d-1+j}{j}\\) | binomial series | [Binomial series](https://en.wikipedia.org/wiki/Binomial_series) |
| 7 | \\(L_m = -\sum i/m\\) over divisors | log of product, Mercator series | [Mercator series](https://en.wikipedia.org/wiki/Mercator_series) |
| 8 | \\(N = \exp(L)\\) | Newton's iteration, power series exp | [cp-algorithms: polynomial ops](https://cp-algorithms.com/algebra/polynomial.html) |
| 9 | \\(f_k = \sum N_j D_{k-j}\\) | polynomial convolution | [cp-algorithms: FFT/NTT](https://cp-algorithms.com/algebra/fft.html) |

{: .box-success}
**Complexity:** \\(O(K \log(nK))\\), where \\(K = \min(M, 250000)\\) and \\(M = \binom{n}{2} - \binom{d}{2}\\) — the sieve for `L` costs \\(O(K \log n)\\) (harmonic sum), the Newton-iteration `exp` costs \\(O(K \log K)\\), and the final convolution costs \\(O(K \log K)\\).
