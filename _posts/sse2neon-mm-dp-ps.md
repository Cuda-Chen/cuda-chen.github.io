---
layout: post
title:
category:
tags:
---

__m128 _mm_dp_ps(__m128 a, __m128 b, const int imm) {

    float s = 0;
    float32x4_t f32a = vreinterpretq_f32_m128(a);
    float32x4_t f32b = vreinterpretq_f32_m128(b);

    if(imm & (1 << 4))
        s += f32a[0] * f32b[0];
    if(imm & (1 << 5))
        s += f32a[1] * f32b[1];
    if(imm & (1 << 6))
        s += f32a[2] * f32b[2];
    if(imm & (1 << 7))
        s += f32a[3] * f32b[3];

    float32x4_t res = {
        (imm & 0x1) ? s : 0,
        (imm & 0x2) ? s : 0,
        (imm & 0x4) ? s : 0,
        (imm & 0x8) ? s : 0,
    };
    return vreinterpretq_m128_f32(res);
}

## ARM64 gcc 10.3

### -O0
_mm_dp_ps(__Float32x4_t, __Float32x4_t, int):
        sub     sp, sp, #112
        str     q0, [sp, 32]
        str     q1, [sp, 16]
        str     w0, [sp, 12]
        str     wzr, [sp, 108]
        ldr     q0, [sp, 32]
        str     q0, [sp, 64]
        ldr     q0, [sp, 16]
        str     q0, [sp, 48]
        ldr     w0, [sp, 12]
        and     w0, w0, 16
        cmp     w0, 0
        beq     .L2
        ldr     s1, [sp, 64]
        ldr     s0, [sp, 48]
        fmul    s0, s1, s0
        ldr     s1, [sp, 108]
        fadd    s0, s1, s0
        str     s0, [sp, 108]
.L2:
        ldr     w0, [sp, 12]
        and     w0, w0, 32
        cmp     w0, 0
        beq     .L3
        ldr     s1, [sp, 68]
        ldr     s0, [sp, 52]
        fmul    s0, s1, s0
        ldr     s1, [sp, 108]
        fadd    s0, s1, s0
        str     s0, [sp, 108]
.L3:
        ldr     w0, [sp, 12]
        and     w0, w0, 64
        cmp     w0, 0
        beq     .L4
        ldr     s1, [sp, 72]
        ldr     s0, [sp, 56]
        fmul    s0, s1, s0
        ldr     s1, [sp, 108]
        fadd    s0, s1, s0
        str     s0, [sp, 108]
.L4:
        ldr     w0, [sp, 12]
        and     w0, w0, 128
        cmp     w0, 0
        beq     .L5
        ldr     s1, [sp, 76]
        ldr     s0, [sp, 60]
        fmul    s0, s1, s0
        ldr     s1, [sp, 108]
        fadd    s0, s1, s0
        str     s0, [sp, 108]
.L5:
        movi    v0.4s, 0
        str     q0, [sp, 80]
        ldr     w0, [sp, 12]
        and     w0, w0, 1
        cmp     w0, 0
        beq     .L6
        ldr     s0, [sp, 108]
        b       .L7
.L6:
        movi    v0.2s, #0
.L7:
        ldr     w0, [sp, 12]
        and     w0, w0, 2
        cmp     w0, 0
        beq     .L8
        ldr     s3, [sp, 108]
        b       .L9
.L8:
        movi    v3.2s, #0
.L9:
        ldr     w0, [sp, 12]
        and     w0, w0, 4
        cmp     w0, 0
        beq     .L10
        ldr     s2, [sp, 108]
        b       .L11
.L10:
        movi    v2.2s, #0
.L11:
        ldr     w0, [sp, 12]
        and     w0, w0, 8
        cmp     w0, 0
        beq     .L12
        ldr     s1, [sp, 108]
        b       .L13
.L12:
        movi    v1.2s, #0
.L13:
        ins     v0.s[1], v3.s[0]
        ins     v0.s[2], v2.s[0]
        ins     v0.s[3], v1.s[0]
        str     q0, [sp, 80]
        ldr     q0, [sp, 80]
        add     sp, sp, 112
        ret

### -O2
_mm_dp_ps(__Float32x4_t, __Float32x4_t, int):
        movi    v2.2s, #0
        tbz     x0, 4, .L2
        dup     s4, v0.s[0]
        dup     s3, v1.s[0]
        fmadd   s2, s4, s3, s2
.L2:
        tbz     x0, 5, .L3
        dup     s3, v0.s[1]
        dup     s4, v1.s[1]
        fmadd   s2, s3, s4, s2
.L3:
        tbz     x0, 6, .L4
        dup     s3, v0.s[2]
        dup     s4, v1.s[2]
        fmadd   s2, s3, s4, s2
.L4:
        tbz     x0, 7, .L5
        dup     s0, v0.s[3]
        dup     s1, v1.s[3]
        fmadd   s2, s0, s1, s2
.L5:
        movi    v1.2s, #0
        tst     x0, 1
        fcsel   s0, s2, s1, ne
        tst     x0, 2
        fcsel   s4, s2, s1, ne
        tst     x0, 4
        fcsel   s3, s2, s1, ne
        tst     x0, 8
        ins     v0.s[1], v4.s[0]
        fcsel   s2, s2, s1, ne
        ins     v0.s[2], v3.s[0]
        ins     v0.s[3], v2.s[0]
        ret

## ARM32 gcc 10.3

### -O0
_mm_dp_ps(__simd128_float32_t, __simd128_float32_t, int):
        push    {r7}
        sub     sp, sp, #116
        add     r7, sp, #0
        vstr    d0, [r7, #40]
        vstr    d1, [r7, #48]
        vstr    d2, [r7, #24]
        vstr    d3, [r7, #32]
        str     r0, [r7, #20]
        mov     r3, #0
        str     r3, [r7, #108]    @ float
        vldr    d16, [r7, #40]
        vldr    d17, [r7, #48]
        vstr    d16, [r7, #72]
        vstr    d17, [r7, #80]
        vldr    d16, [r7, #24]
        vldr    d17, [r7, #32]
        vstr    d16, [r7, #56]
        vstr    d17, [r7, #64]
        ldr     r3, [r7, #20]
        and     r3, r3, #16
        cmp     r3, #0
        beq     .L2
        vldr.32 s14, [r7, #72]
        vldr.32 s15, [r7, #56]
        vmul.f32        s15, s14, s15
        vldr.32 s14, [r7, #108]
        vadd.f32        s15, s14, s15
        vstr.32 s15, [r7, #108]
.L2:
        ldr     r3, [r7, #20]
        and     r3, r3, #32
        cmp     r3, #0
        beq     .L3
        vldr.32 s14, [r7, #76]
        vldr.32 s15, [r7, #60]
        vmul.f32        s15, s14, s15
        vldr.32 s14, [r7, #108]
        vadd.f32        s15, s14, s15
        vstr.32 s15, [r7, #108]
.L3:
        ldr     r3, [r7, #20]
        and     r3, r3, #64
        cmp     r3, #0
        beq     .L4
        vldr.32 s14, [r7, #80]
        vldr.32 s15, [r7, #64]
        vmul.f32        s15, s14, s15
        vldr.32 s14, [r7, #108]
        vadd.f32        s15, s14, s15
        vstr.32 s15, [r7, #108]
.L4:
        ldr     r3, [r7, #20]
        and     r3, r3, #128
        cmp     r3, #0
        beq     .L5
        vldr.32 s14, [r7, #84]
        vldr.32 s15, [r7, #68]
        vmul.f32        s15, s14, s15
        vldr.32 s14, [r7, #108]
        vadd.f32        s15, s14, s15
        vstr.32 s15, [r7, #108]
.L5:
        vmov.f32        q8, #0.0  @ v4sf
        vstr    d16, [r7, #88]
        vstr    d17, [r7, #96]
        ldr     r3, [r7, #20]
        and     r3, r3, #1
        cmp     r3, #0
        beq     .L6
        ldr     r0, [r7, #108]    @ float
        b       .L7
.L6:
        mov     r0, #0
.L7:
        ldr     r3, [r7, #20]
        and     r3, r3, #2
        cmp     r3, #0
        beq     .L8
        ldr     r1, [r7, #108]    @ float
        b       .L9
.L8:
        mov     r1, #0
.L9:
        ldr     r3, [r7, #20]
        and     r3, r3, #4
        cmp     r3, #0
        beq     .L10
        ldr     r2, [r7, #108]    @ float
        b       .L11
.L10:
        mov     r2, #0
.L11:
        ldr     r3, [r7, #20]
        and     r3, r3, #8
        cmp     r3, #0
        beq     .L12
        ldr     r3, [r7, #108]    @ float
        b       .L13
.L12:
        mov     r3, #0
.L13:
        str     r0, [r7]  @ float
        str     r1, [r7, #4]      @ float
        str     r2, [r7, #8]      @ float
        str     r3, [r7, #12]     @ float
        vld1.64 {d16-d17}, [r7:64]
        vstr    d16, [r7, #88]
        vstr    d17, [r7, #96]
        vldr    d16, [r7, #88]
        vldr    d17, [r7, #96]
        vmov    q0, q8  @ v4sf
        adds    r7, r7, #116
        mov     sp, r7
        ldr     r7, [sp], #4
        bx      lr

### -O2
_mm_dp_ps(__simd128_float32_t, __simd128_float32_t, int):
        sub     sp, sp, #16
        vldr.32 s15, .L25
        lsls    r3, r0, #27
        bpl     .L2
        vmov.32 r3, d0[0]
        vmov    s13, r3
        vmov.32 r3, d2[0]
        vmov    s14, r3
        vmla.f32        s15, s13, s14
.L2:
        lsls    r1, r0, #26
        bpl     .L3
        vmov.32 r3, d2[1]
        vmov    s13, r3
        vmov.32 r3, d0[1]
        vmov    s14, r3
        vmla.f32        s15, s14, s13
.L3:
        lsls    r2, r0, #25
        bpl     .L4
        vmov.32 r3, d3[0]
        vmov    s13, r3
        vmov.32 r3, d1[0]
        vmov    s14, r3
        vmla.f32        s15, s14, s13
.L4:
        lsls    r3, r0, #24
        bpl     .L5
        vmov.32 r3, d1[1]
        vmov    s13, r3
        vmov.32 r3, d3[1]
        vmov    s14, r3
        vmla.f32        s15, s13, s14
.L5:
        vldr.32 s14, .L25
        tst     r0, #1
        ite     eq
        vmoveq.f32      s13, s14
        vmovne.f32      s13, s15
        tst     r0, #2
        ite     eq
        vmoveq.f32      s12, s14
        vmovne.f32      s12, s15
        tst     r0, #4
        vstr.32 s13, [sp]
        ite     eq
        vmoveq.f32      s13, s14
        vmovne.f32      s13, s15
        tst     r0, #8
        vstr.32 s12, [sp, #4]
        it      eq
        vmoveq.f32      s15, s14
        vstr.32 s13, [sp, #8]
        vstr.32 s15, [sp, #12]
        vld1.64 {d0-d1}, [sp:64]
        add     sp, sp, #16
        bx      lr
.L25:
        .word   0

## Time Comparison
- sse2neon test suite 
- select random five consecutive runs
- std::chrono high resolution clock
### before
armv8:
_mm_dp_ps conversion2.02e-06                                                       
_mm_dp_ps conversion2.037e-06                                                      
_mm_dp_ps conversion1.885e-06                                                      
_mm_dp_ps conversion1.639e-06                                                      
_mm_dp_ps conversion1.501e-06
1.8164e-06 
armv7: 
_mm_dp_ps conversion2.73e-06                                                       
_mm_dp_ps conversion2.497e-06                                                      
_mm_dp_ps conversion2.749e-06                                                      
_mm_dp_ps conversion2.54e-06                                                       
_mm_dp_ps conversion2.838e-06 
2.6708e-06

### after
armv8:
_mm_dp_ps conversion1.912e-06                                                      
_mm_dp_ps conversion2.196e-06                                                      
_mm_dp_ps conversion1.938e-06                                                      
_mm_dp_ps conversion2.029e-06
_mm_dp_ps conversion1.726e-06
1.9606e-06
armv7:
_mm_dp_ps conversion2.12e-06                                                       
_mm_dp_ps conversion1.97e-06                                                       
_mm_dp_ps conversion1.952e-06                                                      
_mm_dp_ps conversion2.001e-06                                                      
_mm_dp_ps conversion1.907e-06
1.828e-06
