---
layout: post
title: "Simplicity Acts Well: Gaining 25% Shorter Latency on Armv7-A"
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

For the base algorithm it utilize VSRA (Vector Shift Right by immediate value and Accumulate) [^6], 
which has at most four latency cycles on Armv7-A [^8].

### Apply VPADD

> The whole optimization is hosted on [^2].

We can use VPADD (Vector Pairwise Add) [^7], 
which has at most three latency cycles mentioned in [^5],
and a mask for a set of powers of 2 to first "shift" the MSB of each element
(recall the maskout value is either `0x00` or `0xff`, and what we
interest is the final integer with each bit position corresponds to 
the index of each element in a register) then accumulate.

Below shows the flow of applying VPADD instruction in our scenario:

```
=========================================================================================
EXAMPLE INPUT (Hex):
[ 80 | 00 | 81 | 00 | 00 | FF | 00 | C0 || 01 | 7F | 82 | 00 | 00 | 00 | 00 | 80 ]
=========================================================================================

-----------------------------------------------------------------------------------------
STEP 1: vshrq_n_s8 (Arithmetic Shift Right by 7)
-----------------------------------------------------------------------------------------
The arithmetic shift duplicates the MSB across all 8 bits of each lane.
If MSB is 1 -> 0xFF. If MSB is 0 -> 0x00.

MSB    :    1    0    1    0    0    1    0    1 ||    0    0    1    0    0    0    0    1
mask   : [ FF | 00 | FF | 00 | 00 | FF | 00 | FF || 00 | 00 | FF | 00 | 00 | 00 | 00 | FF ]

-----------------------------------------------------------------------------------------
STEP 2: vandq_u8 (Bitwise AND with Powers of 2)
-----------------------------------------------------------------------------------------
Isolates a specific bit-weight for each lane to prepare for summation.

w      : [  1 |  2 |  4 |  8 | 16 | 32 | 64 |128 ||  1 |  2 |  4 |  8 | 16 | 32 | 64 |128 ]
mask   : [ FF | 00 | FF | 00 | 00 | FF | 00 | FF || 00 | 00 | FF | 00 | 00 | 00 | 00 | FF ]
weighted:[  1 |  0 |  4 |  0 |  0 | 32 |  0 |128 ||  0 |  0 |  4 |  0 |  0 |  0 |  0 |128 ]
         \_______________ LOW HALF ______________/  \_______________ HIGH HALF _____________/

-----------------------------------------------------------------------------------------
STEP 3: vpadd_u8 (Horizontal Pairwise Addition)
-----------------------------------------------------------------------------------------
Accumulates the weights. Each `vpadd_u8` halves the number of elements by adding adjacent pairs.

1st vpadd_u8 (low, high): Adds adjacent pairs in LOW, then adjacent pairs in HIGH.
low sums : (1+0)=1, (4+0)=4, (0+32)=32, (0+128)=128
high sums: (0+0)=0, (4+0)=4, (0+0)=0, (0+128)=128
p =      [  1 |  4 | 32 |128 ||  0 |  4 |  0 |128 ]

2nd vpadd_u8 (p, p): Adds adjacent pairs of the new vector 'p'.
sums     : (1+4)=5, (32+128)=160, (0+4)=4, (0+128)=128 ... (repeated for high)
p =      [  5 | 160|  4 | 128||  5 | 160|  4 | 128]

3rd vpadd_u8 (p, p): Adds adjacent pairs one last time.
sums     : (5+160)=165, (4+128)=132 ... (repeated)
p =      [ 165| 132| 165| 132|| 165| 132| 165| 132]

(Note: 165 in hex is 0xA5. 132 in hex is 0x84).
p (Hex) = [ A5 | 84 | A5 | 84 | A5 | 84 | A5 | 84 ]

-----------------------------------------------------------------------------------------
STEP 4: vget_lane_u16 (Extract the 16-bit integer)
-----------------------------------------------------------------------------------------
Reinterprets the vector as 16-bit integers and extracts the 0th lane (the first two bytes).
Because ARM is little-endian, byte 0 (A5) is the lower byte, and byte 1 (84) is the upper.

Extracted Hex = 0x84A5

Binary Result:
  1000 0100   1010 0101
  \_______/   \_______/
   High MSBs   Low MSBs

```

The reference implementation is shown here:

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

By this implementation, almost all the benchmarks improve significantly.
The drawback is that if the matched element resides in the
first round, it results in inferior performance (about at most 13% degration
of execution time) as this optimization needs to load the shift mask into register.

The entire benchmark table is depicted as follows:

```
    +---------------------------+--------------------+--------------------+--------------+
    | Benchmark                 | Current (ns)       | This PR (ns)       | Speedup Rate |
    +---------------------------+--------------------+--------------------+--------------+
    | BM_Throughput_AllZero     | 2.6525282890229187 | 2.4566457585583303 | 7.9735766%   |
    | BM_Throughput_AllOnes     | 2.8733532406965994 | 2.4557148112294884 | 17.0067969%  |
    | BM_Throughput_Alternating |  3.342313203681577 | 2.4557671038580566 | 36.1005772%  |
    | BM_Throughput_CmpResult   | 2.6522269210890794 | 2.4555904849335204 | 8.0077048%   |
    | BM_Throughput_Random      | 2.9464822592339894 | 2.4562281095118546 | 19.9596344%  |
    | BM_Latency                |  8.103782835634545 |  6.482648720958982 | 25.00728%    |
    | BM_Memchr_FoundAt2048     |  266.1089131830451 | 219.28191142185295 | 21.3547034%  |
    | BM_Memchr_NotFound        |  528.9246602358438 |  441.7387922789431 | 19.7369734%  |
    | BM_Memchr_FoundAt0        | 2.5823763925921757 | 2.9466702218510674 | -12.3628978% |
    +---------------------------+--------------------+--------------------+--------------+
```

## Closing Thoughts

In this post, I take a brief history of string pattern matching. Then,
I introduction `_mm_movemask_epi8()`, the swiss-knife of constructing
the occurence indices of a certain pattern.
Next, I propose my implementation with a persuasive benchmark result
on almost the pattern matching case. Ultimately, the implementation
is recorded in the world-class sse2neon project, leveraging all Armv7-A
platform with better performance and saving the power consumption.

As there are a lot of types of optimizing string pattern matching, leave
your comments for the further discussion of your own method!

## References

[^1]: https://github.com/DLTcollab/sse2neon/pull/768

[^2]: https://github.com/DLTcollab/sse2neon/pull/769

[^3]: https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#text=_mm_movemask_epi8

[^4]: https://en.wikipedia.org/wiki/Hamming_weight

[^5]: https://support.arm.com/documentation/ddi0409/h/Instruction-Timing/Instruction-specific-scheduling/Advanced-SIMD-integer-arithmetic-instructions?lang=en 

[^6]: https://support.arm.com/documentation/dui0473/k/neon-and-vfp-instructions/vsra--by-immediate-?lang=en

[^7]: https://support.arm.com/documentation/ddi0406/b/Application-Level-Architecture/Instruction-Details/Alphabetical-list-of-instructions/VPADD--integer-?lang=en

[^8]: https://support.arm.com/documentation/ddi0409/h/Instruction-Timing/Instruction-specific-scheduling/Advanced-SIMD-integer-shift-instructions?lang=en
