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
