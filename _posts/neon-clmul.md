---
layout: post
title: neon clmul
category:
tags:
---

## CRC32C

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
