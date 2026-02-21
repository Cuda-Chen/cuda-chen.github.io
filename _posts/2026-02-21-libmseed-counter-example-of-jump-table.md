---
layout: post
title: libmseed Optimization Attempt -- A Counter Example of Replacing switch-case
category: [programming, seismology, seismology data format, optimziation, benchmarking]
tags: [programmers, C, miniSEED, libmseed, benchmarking, optimization]
---

## Outline

For system software programmers, we always thrive
for any possibilities of performance optimization.
When it comes to branching, we believes
that a solely jump table, which does a certain operation
after a arithmetic operation for calculating the entry
of the jump table, beats many methods
including long if-else and complex switch-case statements.

In this post, I am going to give you a counter example
that a seems-dumb switch-case statement beats
jump table technique. I will not only show the benchmark
result, but also some personal findings that
I think why a switch-case statement become the better
performant one compared to the jump table implementation.

## The Target

I am going to do an optimization of `msr_decode_steim2()`
in a widely-used siesmic data library called libmseed.

If you are interested in the format of miniSEED with Steim2 format,
check [^2].

You can view the whole part of `msr_decode_steim2()` in [^1].
For your clarity, here I provide a minified version of the code:

```c
int64 msr_decode_steim2(...) {
    int nibble = ...; /* check the first nibble */
    int dnib; /* second nibble placeholder */    

    switch(nibble) {
        case 0:
            break;
        case 1:
            break;
        case 2:
            dnib = ...; /* check the second nibble */
            switch(dnib) {
                case 0:
                    break;
                case 1:
                    break;
                case 2:
                    break;
                case 3:
                    break;
            }
            break;
        case 3:
            dnib = ...;
            switch(dnib) {
                case 0:
                    break;
                case 1:
                    break;
                case 2:
                    break;
                case 3:
                    break;
            }
            break;
    }

    return 0;
}
```

### What I am Going to Do

Eliminate the non-trivial switch-case statment. Moreover,
check any possibilities for micro-optimization.

## Attempts

For the goal, I come up with these methods: jump table
and for loop.

### jump table

For dozens of conditions, we can use a jump table so that
we let the program to execute certain actions based on the
value of each condition. Usually, this can yield with a
better performances compared to switch-case statement. 

We can just move the statement of each switch-case
into a series of functions, then create a table for
indexing the actions:

```c
/* a series of callbacks */
int fnoop(...) {...} /* nibble=0x00 */
int f01(...) {...} /* nibble=0x01 */
int f1010(...) {...} /* nibble=0x02 and dnib=0x02
...

typedef int (*steim2_decode_func_cb) (uint32_t , /* input frame */
                                      int32_t *,  /* output difference array */
                                      int *      /* output difference array index */
); 
static steim2_decode_func_cb __steim2_decode_func_tbl[16] = {
    fnoop, fnoop, fnoop, fnoop, f01,   f01,   f01,   f01, 
    f1000, f1001, f1010, f1011, f1100, f1101, f1110, f1111,
};

int64_t msr_decode_steim2(...) {
    /* Substitute the swtich-case into array indexing */
    int nibble = ...;
    int dnib = ...; /* Always check the second nibble */
    uint32_t ii = ((nibble & 0x03) << 2) | (dnib & 0x03);

    steim2_decode_func_cb handler = __steim2_decode_func_tbl[ii & 0x0f];
    int ret = handler (frame[widx], diff, &diffidx);
    ...
}
```

### for loop

As specified in [^2], we can use a for loop with precomputed
values to determined how many bits should be scanned each time
and how many values stored in a frame.

So it will become like this:

```c
int64_t msr_decode_steim2(...) {
      /* Substitute the swtich-case into array indexing */
      int nibble = ...;
      int dnib = ...; /* Always check the second nibble */
      uint32_t ii = ((nibble & 0x03) << 2) | (dnib & 0x03);

      int base[4] = {0, 4, 0, 1,};
      uint32_t increment_mask[4] = {0x0, 0x0, 0x07, 0x07,};
      int cnt = base[nibble & 0x03] + (ii & increment_mask[nibble & 0x03]);

      /* start bit of each combination of nibble and dnib */
      int start_bit_pos[16] = {
          0, 0, 0, 0,
          24, 24, 24, 24,
          0, 0, 15, 20,
          24, 25, 24, 0, 
      };

      /* bit length of each combination of nibble and dnib */
      int bb[16] = { 
          0, 0, 0, 0,
          8, 8, 8, 8,
          0, 30, 15, 10,
          6, 5, 4, 0,
      };

      for (idx = 0; idx < cnt; idx++)
      {        
        int bit_count = bb[ii & 0x0f];
        int shift = 32 - bit_count;
        /* The nibble=0x01 needs extra treatment as there are no
        * any defintion of little-endian Steim2 SEED.
        * See https://github.com/EarthScope/libmseed/issues/36#issuecomment-470370790
        * for more details.
        */
        int start = start_bit_pos[ii & 0x0f] - (
                nibble == 0x01
                ? cnt - idx - 1 
                : idx) * bit_count;

        /* adapted from "Sign extending from a variable bit-width" section in https://graphics.stanford.edu/~seander/bithacks.html */
        int32_t m = 1U << (bit_count - 1); 
        int32_t t = (((frame[widx]) >> (start)) & ((1U << (bit_count)) - 1));
        int32_t tmp = (t ^ m) - m;

        diff[diffidx++] = tmp; 
      }
}
```

## Benchmark

For benchmarking, out goal is to reduce the execution time.

### environment

- OS: Linux v6.8.0
- CPU: Intel(R) Core(TM) i5-3230M CPU @ 2.60GHz
- compiler flags: `-O2`
- compiler: GCC 13.3.0

### benchmark program

- decode_rodeo
  - download from [^3]
  - run 1000000 iterations on Steim2 encoded input data

The sample output of benchmark program:

```
$ ./decode_rodeo 
Testing Steim-2
reclen: 1595
dataoffset: 59
datasize: 1536
Steim2 decoded 1000000 iterations in 2.045135 seconds
```

### comparison

| type | execution time (measured in seconds) | performance gain (compared to baseline) |
| --- | --- | --- | 
| baseline | 2.045135 | 0.0 % |
| jump table | 2.451932 | -19.8909607 % |
| for loop | 3.792999 | -85.4644803 % |

## Findings

We can realize that all of the attempts are signicantly slower than
baseline.

Though the lines of disassembly of each attempts are fewer,
we get the inferior result.

After some investigations, I conclude why any attempts result in
inferior result:
- For jump table, it requires to make a function call, which
becomes a big impact on decoding procedure.
- For for loop method, we calculate the needed values after
we get the nibbles when decoding each frame. In fact, this
part become the bottleneck of the decoding process.

## Recap

- If you are certain the action of each condition, consider
using code templating (e.g., macros) rather than function calls.
- Benchmark, benchmark, and benchmark. 

## References

[^1]: https://github.com/EarthScope/libmseed/blob/07a6e2d8b4611e9f59155d5632ab24dc2598cf9f/unpackdata.c#L356 
[^2]: https://www.fdsn.org/pdf/SEEDManual_V2.4.pdf

[^3]: https://github.com/EarthScope/libmseed/pull/102#issuecomment-1614927016
