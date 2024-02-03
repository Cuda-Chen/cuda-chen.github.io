---
layout: post
title: neon clmul
category:
tags:
---

## Introduction

In this post, I am going to illustrate the progress of `_mm_crc32_u8`
conversion improvement of the contribution to sse2neon.

In the beginning, I will make a brief introduction to CRC32C,
which is the CRC algorithm that `_mm_crc32_u8` applies.
Then, I will show how I optimize the conversion with various method.

## What's CRC32C?

Before explaining CRC32C, I would like to answer a question: what
is CRC? It is an algorithm used for error detection in network and storage device [^1],
and its entire name is Cyclic Redundancy Check.
The sender uses a number as divisor, then applies division on the message
to get the remainder. Next, sender appends the remainder in the end
of the message. To verify whether the message has any errors,
the receiver applies division on the message. If the remainder
is not zero, it means the message is errorous. As it doesn't modify
the content of message (redundancy) and the division is just shifting
the divident then subtract (cyclic code), so the name, CRC,
represents these behaviors.

A CRC algorithm is called an n-bit CRC when its divisor (formally
check value) is n-bit long. Thus, the CRC32C, a variant of CRC32, has 
a 32-bit binary number as the dividend.

## carry-less multiplication

## Road of optimization

```c
crc ^= v;
for (int bit = 0; bit < 8; bit++)
    crc = (crc & 1) ? ((crc >> 1) ^ UINT32_C(0x82f63b78)) : (crc >> 1);
```

not good as mentioned in https://github.com/DLTcollab/sse2neon/pull/627#discussion_r1453360563

## Table method

```c
```

Reviewer request not to use this as it costs 1KiB space.

## Table method (half-byte)

```c
```

Just need 64B space!

## Barrett Reduction

- https://mary.rs/lab/crc32/
- multiplicative inverse
    - https://github.com/calccrypto/uint256_t

## CRC notation

- reverse (bit-reflected)
- Reciprocal

## Conclusion

## Trivia

- It is confirmed that the precedence of test function will affect the running time of them in qemu.

## Reference

[^1] https://en.wikipedia.org/wiki/Cyclic_redundancy_check
