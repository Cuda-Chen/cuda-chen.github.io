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

Months ago, I received a PR [^1] related to `_mm_movemask_epi8()`
[^3], one of the famous intrinsic that usually applies on string pattern
matching. The PR claimed to have better
performance by copying high bytes directly and extract the
16-bit result, reducing assembly instructions from seven to six.
Nevertheless, it got inferior performance on A32 platform.

After checking the code, I suspected that I can replace the
`vsraq_n_u64()` to other instructions with similiar effect
whilst has fewer instruction latency, boosting the performance
furthermore.

## Check the Occurences of A Character

Suppose we are going to find the indices of an ASCII-encoded character `'#'`
in a string `"c#dach#nblog#ere"`, we can use the following steps to
efficiently find the indices:

1. Fill a MMX register (`a`) with eight `'#'` (0x23).
2. Load the string into another MMX register (`b`) with `"c#dach#nblog#ere"`.
3. Perform elementwise AND between `a` and `b`.
4. Retrieve the MSB of each element, then store into an integer.
5. Count the indicies of one (usually you can use popcnt, formally
Hamming weight [^4] with better performance). 

For `_mm_movemask_epi8()` it merges the whole steps from 1. to 5.,
yielding better performance with hardware integration and ease of use:

> int _mm_movemask_epi8 (__m128i a)
> Create mask from the most significant bit of each 8-bit element in a, and store the result in dst. 

## My Optimization

### Base

### Apply VPADD

## Closing Thoughts

## References

[^1]: https://github.com/DLTcollab/sse2neon/pull/768

[^2]: https://github.com/DLTcollab/sse2neon/pull/769

[^3]: https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#text=_mm_movemask_epi8

[^4]:  https://en.wikipedia.org/wiki/Hamming_weight
