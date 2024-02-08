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

As a reminder, the CRC32C uses the following polynominals:

- normal: 0x1EDC6F41
- bit-reflected: 0x82F63B78

What's more, we use the bit-reflected way for implementation.
For the reasons of using bit-reflected method,
you can refer to Fastest CRC32 for x86, Intel and AMD, + comprehensive derivation and discussion of various approaches [^2].

## Road of optimization

Let's start with the original implementation in sse2neon [^3]:

```c
FORCE_INLINE uint32_t _mm_crc32_u8(uint32_t crc, uint8_t v)
{
    // If the target does not support CRC extension
    crc ^= v;
    for (int bit = 0; bit < 8; bit++) {
        if (crc & 1)
            crc = (crc >> 1) ^ UINT32_C(0x82f63b78);
        else
            crc = (crc >> 1);
    }
    return crc;
}
```

### Apply ternany operator

Modern compiler can optimize the ternany operator into
conditional move to prevent branching. As a consequence, we can
re-write the `if...else` statement into ternany operator:

```c
for (int bit = 0; bit < 8; bit++)
    crc = (crc & 1) ? ((crc >> 1) ^ UINT32_C(0x82f63b78)) : (crc >> 1);
```

However, as mentioned by the reviewer [^4], we should come up
with another way to utilize the power of NEON.

## Table method

*describe the byte to T*

```c
```

Reviewer request not to use this as it costs 1KiB space.

## Table method (half-byte)

```c
```

Just need 64B space!

## carry-less multiplication

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

[^2] https://github.com/komrad36/CRC

[^3] https://github.com/DLTcollab/sse2neon/blob/cfaa59fc04fecb117c0a0f3fe9c82dece6f359ad/sse2neon.h#L8502

[^4] https://github.com/DLTcollab/sse2neon/pull/627#discussion_r1453360563
