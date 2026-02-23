---
layout: post
title: ICPC 2022 Regional H, Hardest Problem
cover-img: TODO
thumbnail-img: TODO
share-img: TODO
tags: [ICPC, ACM, competitive programming, algorithms]
author: Binh Tran
---

My most recent competitive programming event was the 2022 ICPC Asia Ho Chi Minh City Regional Contest, where the top four university teams qualified for the World Final. Our team finished in 18th place, making us the fifth highest, having solved one problem fewer than the fourth-ranked team. At the start of the contest, I was drawn to [Problem H](https://oj.vnoi.info/problem/icpc22_regional_h#comments), a math question that was arguably the most challenging of the competition. Confident in my math skills, I believed I could solve it, but I couldn't. After over an hour of struggle, I realized it was too difficult for me and decided to move on. I often think about how if I had set it aside sooner, we might have solved one more other problem and secured a place in the World Final. I took the problem statement home to try again but was still unsuccessful. Although this contest was three years ago, the problem statement remains on my desk. I recently found it while cleaning for the Lunar New Year, and it seems I still haven't moved on!

You can find the problem statement in the link above. TLDR:

Given two integers n and d. Define f(k) as the number of permutations of 1, 2, ..., n such that:
- The number of inversions of the permutation is k.
- When removing all elements with values that are strictly greater than d from the permutation, the remaining elements are sorted in increasing order.

Find f(k) modulo 998244353 for all k.

The solution was never revealed, but it was vaguely discussed in this [post](https://www.facebook.com/code.cung.rr/posts/pfbid02Xue7uxdVBTZqeEMTP7vBewiXQKsXzbK1GBMpX3DTuJRWZB8oRGejdZKHmNWfGyXMl?rdid=DraLgPH36bn3SC1M#). I will try to explain it in detail, so that I can understand :).
