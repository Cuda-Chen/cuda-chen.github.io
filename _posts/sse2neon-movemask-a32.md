---
layout: post
title:
category: [Programming]
tags: [C, C++, sse2neon, Open Source Contribution, A32, Arm, NEON]
---

String matching plays a crucial part nowadays: the goal is to
search the certain patterns of string or character. It is widely
used in networking and text processing.

For decades, we have proposed a variety of methods for optimizations
including utilizing SIMD instructions. In this post, I am going
make an instance of finding all indices of a character in a string.
Then, I will propose my technique of optimization on A32 platform
with reducing latency time at least 25 percent when dealing with
long input string. 

## The Starting Point

Months ago, I received a PR [^1] that claimed to have better
performance by copying high bytes directly and extract the
16-bit result, reducing assembly instructions from seven to six.
Nevertheless, it got inferior performance on A32 platform.

After checking the code, I suspected that I can replace the
`vsraq_n_u64()` to other instructions with similiar effect
whilst has fewer instruction latency, boosting the performance
furthermore.

## Check the Occurences of A Character

movemask

## My Optimization

## Closing Thoughts

## References

[^1]: https://github.com/DLTcollab/sse2neon/pull/768

[^2]: https://github.com/DLTcollab/sse2neon/pull/769
