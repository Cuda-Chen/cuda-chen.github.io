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

As a reminder, the CRC32C uses the following polynominals (I will represent
as P for the rest of post):

- normal: 0x1EDC6F41
- bit-reflected: 0x82F63B78

What's more, we use the bit-reflected way for implementation.
For the reasons of using bit-reflected method,
you can refer to *Fastest CRC32 for x86, Intel and AMD, + comprehensive derivation and discussion of various approaches* [^2].

## Road of optimization

Let's start with the original implementation in sse2neon [^3]:

```c
FORCE_INLINE uint32_t _mm_crc32_u8(uint32_t crc, uint8_t v)
{
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

## Tabular method

Observing the following implementation of calculating CRC32C:

 ```c
for (int bit = 0; bit < 8; bit++)
    crc = (crc & 1) ? ((crc >> 1) ^ UINT32_C(0x82f63b78)) : (crc >> 1);
```

You can realize that which bits of P will be shifted in of P then XOR'd are uniquely
deretmined by the rightmost 8 bits of `crc`. Thus, we can rewrite the calculation
procedure as follows:

```c
// I use A, B, C, D, ...
// as the substitution of either 0 or the polynominal.

crc = (crc >> 1) ^ A
crc = (crc >> 1) ^ B
crc = (crc >> 1) ^ C
crc = (crc >> 1) ^ D
crc = (crc >> 1) ^ E
crc = (crc >> 1) ^ F
crc = (crc >> 1) ^ G
crc = (crc >> 1) ^ H
```

We then rewrite the above procedure to a single expression:

```c
(((((((((((((((crc >> 1) ^ A) >> 1) ^ B) >> 1) ^ C) >> 1) ^ D) >> 1) ^ E) >> 1) ^ F) >> 1) ^ G) >> 1) ^ H
```

Re-distribute the shifts for simplifying the expression:

```c
(crc >> 8) ^ (A >> 7) ^ (B >> 6) ^ (C >> 5) ^ (D >> 4) ^ (E >> 3) ^ (F >> 2) ^ (G >> 1) ^ H
```

Then, combine all the terms from A to H into a single value T:

```c
(crc >> 8) ^ T
```

We can precompute the value of T because it is merely composed of
256 permutations (recall that we just do calculation on the rightmost
8 bits): 

```c
// Adopted from qemu: https://github.com/qemu/qemu/blob/907209e3111dd62a553a19319b422ff8aba5b9c0/util/crc32c.c#L40

static const uint32_t _sse2neon_crc32_tbl[] = {
    0x00000000, 0xF26B8303, 0xE13B70F7, 0x1350F3F4,
    0xC79A971F, 0x35F1141C, 0x26A1E7E8, 0xD4CA64EB,
    0x8AD958CF, 0x78B2DBCC, 0x6BE22838, 0x9989AB3B,
    0x4D43CFD0, 0xBF284CD3, 0xAC78BF27, 0x5E133C24,
    0x105EC76F, 0xE235446C, 0xF165B798, 0x030E349B,
    0xD7C45070, 0x25AFD373, 0x36FF2087, 0xC494A384,
    0x9A879FA0, 0x68EC1CA3, 0x7BBCEF57, 0x89D76C54,
    0x5D1D08BF, 0xAF768BBC, 0xBC267848, 0x4E4DFB4B,
    0x20BD8EDE, 0xD2D60DDD, 0xC186FE29, 0x33ED7D2A,
    0xE72719C1, 0x154C9AC2, 0x061C6936, 0xF477EA35,
    0xAA64D611, 0x580F5512, 0x4B5FA6E6, 0xB93425E5,
    0x6DFE410E, 0x9F95C20D, 0x8CC531F9, 0x7EAEB2FA,
    0x30E349B1, 0xC288CAB2, 0xD1D83946, 0x23B3BA45,
    0xF779DEAE, 0x05125DAD, 0x1642AE59, 0xE4292D5A,
    0xBA3A117E, 0x4851927D, 0x5B016189, 0xA96AE28A,
    0x7DA08661, 0x8FCB0562, 0x9C9BF696, 0x6EF07595,
    0x417B1DBC, 0xB3109EBF, 0xA0406D4B, 0x522BEE48,
    0x86E18AA3, 0x748A09A0, 0x67DAFA54, 0x95B17957,
    0xCBA24573, 0x39C9C670, 0x2A993584, 0xD8F2B687,
    0x0C38D26C, 0xFE53516F, 0xED03A29B, 0x1F682198,
    0x5125DAD3, 0xA34E59D0, 0xB01EAA24, 0x42752927,
    0x96BF4DCC, 0x64D4CECF, 0x77843D3B, 0x85EFBE38,
    0xDBFC821C, 0x2997011F, 0x3AC7F2EB, 0xC8AC71E8,
    0x1C661503, 0xEE0D9600, 0xFD5D65F4, 0x0F36E6F7,
    0x61C69362, 0x93AD1061, 0x80FDE395, 0x72966096,
    0xA65C047D, 0x5437877E, 0x4767748A, 0xB50CF789,
    0xEB1FCBAD, 0x197448AE, 0x0A24BB5A, 0xF84F3859,
    0x2C855CB2, 0xDEEEDFB1, 0xCDBE2C45, 0x3FD5AF46,
    0x7198540D, 0x83F3D70E, 0x90A324FA, 0x62C8A7F9,
    0xB602C312, 0x44694011, 0x5739B3E5, 0xA55230E6,
    0xFB410CC2, 0x092A8FC1, 0x1A7A7C35, 0xE811FF36,
    0x3CDB9BDD, 0xCEB018DE, 0xDDE0EB2A, 0x2F8B6829,
    0x82F63B78, 0x709DB87B, 0x63CD4B8F, 0x91A6C88C,
    0x456CAC67, 0xB7072F64, 0xA457DC90, 0x563C5F93,
    0x082F63B7, 0xFA44E0B4, 0xE9141340, 0x1B7F9043,
    0xCFB5F4A8, 0x3DDE77AB, 0x2E8E845F, 0xDCE5075C,
    0x92A8FC17, 0x60C37F14, 0x73938CE0, 0x81F80FE3,
    0x55326B08, 0xA759E80B, 0xB4091BFF, 0x466298FC,
    0x1871A4D8, 0xEA1A27DB, 0xF94AD42F, 0x0B21572C,
    0xDFEB33C7, 0x2D80B0C4, 0x3ED04330, 0xCCBBC033,
    0xA24BB5A6, 0x502036A5, 0x4370C551, 0xB11B4652,
    0x65D122B9, 0x97BAA1BA, 0x84EA524E, 0x7681D14D,
    0x2892ED69, 0xDAF96E6A, 0xC9A99D9E, 0x3BC21E9D,
    0xEF087A76, 0x1D63F975, 0x0E330A81, 0xFC588982,
    0xB21572C9, 0x407EF1CA, 0x532E023E, 0xA145813D,
    0x758FE5D6, 0x87E466D5, 0x94B49521, 0x66DF1622,
    0x38CC2A06, 0xCAA7A905, 0xD9F75AF1, 0x2B9CD9F2,
    0xFF56BD19, 0x0D3D3E1A, 0x1E6DCDEE, 0xEC064EED,
    0xC38D26C4, 0x31E6A5C7, 0x22B65633, 0xD0DDD530,
    0x0417B1DB, 0xF67C32D8, 0xE52CC12C, 0x1747422F,
    0x49547E0B, 0xBB3FFD08, 0xA86F0EFC, 0x5A048DFF,
    0x8ECEE914, 0x7CA56A17, 0x6FF599E3, 0x9D9E1AE0,
    0xD3D3E1AB, 0x21B862A8, 0x32E8915C, 0xC083125F,
    0x144976B4, 0xE622F5B7, 0xF5720643, 0x07198540,
    0x590AB964, 0xAB613A67, 0xB831C993, 0x4A5A4A90,
    0x9E902E7B, 0x6CFBAD78, 0x7FAB5E8C, 0x8DC0DD8F,
    0xE330A81A, 0x115B2B19, 0x020BD8ED, 0xF0605BEE,
    0x24AA3F05, 0xD6C1BC06, 0xC5914FF2, 0x37FACCF1,
    0x69E9F0D5, 0x9B8273D6, 0x88D28022, 0x7AB90321,
    0xAE7367CA, 0x5C18E4C9, 0x4F48173D, 0xBD23943E,
    0xF36E6F75, 0x0105EC76, 0x12551F82, 0xE03E9C81,
    0x34F4F86A, 0xC69F7B69, 0xD5CF889D, 0x27A40B9E,
    0x79B737BA, 0x8BDCB4B9, 0x988C474D, 0x6AE7C44E,
    0xBE2DA0A5, 0x4C4623A6, 0x5F16D052, 0xAD7D5351, 
};

FORCE_INLINE uint32_t _mm_crc32_u8(uint32_t crc, uint8_t v)
{
    crc ^= v;
	crc = (crc >> 8) ^ _sse2neon_crc32_tbl[crc & 0xFF];
    return crc;
}
```

However, reviewer requested not to use this as it costs 1KiB space [^5],
which for my point-of-view, 1KiB space is costly on embedded system such
as Raspberry Pi. Therefore, we have to emerge another tabular method solution
with the balance between performance and space.

### Tabular method (half-byte)

As mentioned in [^6], we can break the whole 8-bit table look-up into two consecutive 4-bit table look-up:

```c
FORCE_INLINE uint32_t _mm_crc32_u8(uint32_t crc, uint8_t v)
{
	crc ^= v;
	static const uint32_t crc32_half_byte_tbl[] = {
	    0x00000000, 0x105ec76f, 0x20bd8ede, 0x30e349b1, 0x417b1dbc, 0x5125dad3,
	    0x61c69362, 0x7198540d, 0x82f63b78, 0x92a8fc17, 0xa24bb5a6, 0xb21572c9,
	    0xc38d26c4, 0xd3d3e1ab, 0xe330a81a, 0xf36e6f75,
	};
	
	crc = (crc >> 4) ^ crc32_half_byte_tbl[crc & 0x0F];
	crc = (crc >> 4) ^ crc32_half_byte_tbl[crc & 0x0F];
	return crc;
}
```

The look-up table just needs to hold every 16th entry of the one-byte tabular method,
thus 16 entries with only 64B space!
As we are implementing `_mm_crc32_u8`, an additional table look-up will
be an afforable compromise.

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

[^5] https://github.com/DLTcollab/sse2neon/pull/627#issuecomment-1895992394

[^6] https://create.stephan-brumme.com/crc32/#half-byte
