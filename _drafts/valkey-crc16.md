---
layout: post
title: "Valkey Contribution Journey: Optimize CRC16 Hash Algorithm"
category: [open source contribution, programming, SIMD]
tags: [programming, C, SIMD, CRC16]
---

## How I Start the Journey

A while ago, I was surfing GitHub to find some issues that I
can help. While glancing the issues in Valkey, I spotted an
issue with a title "[NEW] Accelerate CRC16 with SIMD" [^1].
As I have made a contribution to [sse2neon on CRC32]({% link _posts/2024-02-27-optimize-mm-crc32-u8-conversion-in-sse2neon.md %}).

## What I Get When the Journey Ends

- Tuning performance just like quality assurance: you
implement a set of methods to prove the original method
acts inferior than your implementations. But sometimes,
it turns that the original method is the best.
- Ask loud. Withouth

## References

[^1]: https://github.com/valkey-io/valkey/issues/2031
