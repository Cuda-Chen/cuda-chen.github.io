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
efficiently find the count of such character (taking SSE as example):

1. Fill an XMM register (`a`) with eight `'#'` (0x23).
2. Load the string into another XMM register (`b`) with `"c#dach#nblog#ere"`.
3. Perform elementwise Equal between `a` and `b`.
4. Retrieve the MSB of each element, then store into an integer.
5. Count the nubmer of ones (usually you can use popcnt, formally
Hamming weight [^4] with better performance). 

The flow graph shows the processes:

```
=================================================================================================
[STEP 1] Fill XMM Register `a` (128-bit)
=================================================================================================
 Char:  '#'  '#'  '#'  '#'  '#'  '#'  '#'  '#'  '#'  '#'  '#'  '#'  '#'  '#'  '#'  '#' 
 Hex:    23   23   23   23   23   23   23   23   23   23   23   23   23   23   23   23
         |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
         v    v    v    v    v    v    v    v    v    v    v    v    v    v    v    v

=================================================================================================
[STEP 2] Load String into XMM Register `b` (128-bit)
=================================================================================================
 Char:  'c'  '#'  'd'  'a'  'c'  'h'  '#'  'n'  'b'  'l'  'o'  'g'  '#'  'e'  'r'  'e'
 Hex:    63   23   64   61   63   68   23   6E   62   6C   6F   67   23   65   72   65
         |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
         v    v    v    v    v    v    v    v    v    v    v    v    v    v    v    v

=================================================================================================
[STEP 3] Elementwise Equal (`a` == `b`)
=================================================================================================
 Match?  No  Yes   No   No   No   No  Yes   No   No   No   No   No  Yes   No   No   No
 Hex:    00   FF   00   00   00   00   FF   00   00   00   00   00   FF   00   00   00
 MSB:    0    1    0    0    0    0    1    0    0    0    0    0    1    0    0    0
         |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
         +----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+
                                            |
=================================================================================================
[STEP 4] Retrieve MSB to Integer (e.g., pmovmskb)
=================================================================================================
                                            v
                          Integer Mask: 0b0001000001000010  (Hex: 0x1042)
                                            |
=================================================================================================
[STEP 5] Hamming Weight (popcnt)
=================================================================================================
                                            v
                              POPCNT(0001000001000010) = 3
=================================================================================================
```

For `_mm_movemask_epi8()` it merges the whole steps from 1. to 4.,
yielding better performance with hardware integration and ease of use:

> int _mm_movemask_epi8 (__m128i a)
> Create mask from the most significant bit of each 8-bit element in a, and store the result in dst. 

## My Optimization

### Base

```c
uint8x16_t msbs = vshrq_n_u8(input, 7);
uint64x2_t bits = vreinterpretq_u64_u8(msbs);
bits = vsraq_n_u64(bits, bits, 7);
bits = vsraq_n_u64(bits, bits, 14);
bits = vsraq_n_u64(bits, bits, 28);
uint8x16_t output = vreinterpretq_u8_u64(bits);
return vgetq_lane_u8(output, 0) | (vgetq_lane_u8(output, 8) << 8);
```

For the base algorithm it utilize VSRA [^6], which has five latency cycles
on Armv7-A [^5].

### Apply VPADD

> The whole optimization is hosted on [^2].

Shown in [^5], we can use VPADD [^7] and a mask for a set of powers of 2
to "shift" the MSB of each element .

```c
// Step 1: Extract MSB of each byte as 0x00 or 0xFF
int8x16_t mask = vshrq_n_s8(vreinterpretq_s8_u8(input), 7);
// Step 2: Apply powers of 2 (1, 2, 4, 8, 16, 32, 64, 128)
static const uint8_t w[16] = {1, 2, 4, 8, 16, 32, 64, 128,
                                  1, 2, 4, 8, 16, 32, 64, 128}; 
uint8x16_t weighted = vandq_u8(vreinterpretq_u8_s8(mask), vld1q_u8(w));
// Step 3: Pairwise add to accumulate the bits
uint8x8_t p = vpadd_u8(vget_low_u8(weighted), vget_high_u8(weighted));
p = vpadd_u8(p, p);
p = vpadd_u8(p, p);
// Step 4: Extract the 16-bit mask
return vget_lane_u16(vreinterpret_u16_u8(p), 0);
```

For the drawback is that if the matched element resides in the
first round, it results in inferior performance (about 13% degration)
as this optimization needs to load the shift mask once.

## Closing Thoughts

## References

[^1]: https://github.com/DLTcollab/sse2neon/pull/768

[^2]: https://github.com/DLTcollab/sse2neon/pull/769

[^3]: https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#text=_mm_movemask_epi8

[^4]: https://en.wikipedia.org/wiki/Hamming_weight

[^5]: 

[^6]: https://support.arm.com/documentation/dui0473/k/neon-and-vfp-instructions/vsra--by-immediate-?lang=en

[^7]: https://support.arm.com/documentation/ddi0406/b/Application-Level-Architecture/Instruction-Details/Alphabetical-list-of-instructions/VPADD--integer-?lang=en
