---
layout: post
title: "My sse2neon Contribution of _rdtsc"
category: [programming, open source contributions]
tags: [programming, C, C++, sse2neon]
---

In this post, I am going to illustrate the path of `_rdtsc` [^1] conversion contribution
on sse2neon. At first, I will introduce the usage of `_rdtsc`, then talk about
the implementation and test case .
The full implementation can be seen in here: https://github.com/DLTcollab/sse2neon/pull/532

## What's `_rdtsc`

The `_rdtsc` is an SSE intrinsic which gets the current timestamp from processor.
The way which makes it special is that it gets the timestamp directly from
hardware, which is suitable for measuring precise execution time.

## Implementations

As this post is talking about the conversion, let's talk about how I implement
the conversions on each target.

### ARMv8-A

Pretty straightforward. You just read the value from CNTVCT_EL0 (counter-timer virtual count register).

```c
uint64_t val;
__asm__ __volatile__("mrs %0, cntvct_el0" : "=r"(val));
return val;
```

### ARMv7-A

The ARMv7-A counterpart is trickier as it has no CNTVCT_EL0. Instead, you can get
the value from PMCCNTR (performance monitors cycle count register).

Nevertheless, PMCCNTR can be accessed only in one of the following conditions:
1. All modes executing at PL1 or higher.
2. User mode when PMCUSERENR.EN == 1 (PMCUSERENR stands for performance monitors user enable register).

What's more, PMCCNTR starts to count only if PMCNTENSET (performance monitors count enable set register)
is set.

If none of the three above conditions is met, you will not be able to access PMCCNTR
or get its value. In fact, you can get the current timestamp using Linux kernel API (`gettimeofday`)
as usually the API is running in kernel mode.

```c
uint32_t pmccntr, pmuseren, pmcntenset;
__asm__ __volatile__("mrc p15, 0, %0, c9, c14, 0" : "=r"(pmuseren));
if (pmuseren & 1) {  
    __asm__ __volatile__("mrc p15, 0, %0, c9, c12, 1" : "=r"(pmcntenset));
    if (pmcntenset & 0x80000000UL) { 
        __asm__ __volatile__("mrc p15, 0, %0, c9, c13, 0" : "=r"(pmccntr));
        return (uint64_t) (pmccntr) << 6;
    }
}

struct timeval tv;
gettimeofday(&tv, NULL);
return (uint64_t) (tv.tv_sec) * 1000000 + tv.tv_usec;
```

## Test Cases

In order to prove the implementation works, I add a dedicated test case for unit testing.

```c
// test case implementation
result_t test_rdtsc(const SSE2NEONTestImpl &impl, uint32_t iter)
{
    uint64_t start = _rdtsc();
    for (int i = 0; i < 100000; i++)
        __asm__ __volatile__("" ::: "memory");
    uint64_t end = _rdtsc();
    return end > start ? TEST_SUCCESS : TEST_FAIL;
}
```

The test procedure as follows: 
1. get current timestamp
2. create a long-running time for loop
3. get current timestamp again
4. check whether the value of timestamp in 3. is larger than 1.

### Why the for loop looks so strange?
You may ask why not create the long-running for loop as follows:

```c
for(int i = 0; i < 100000; i++)
    ; // no-op
```

The reason is that modern compile sometimes eliminates the loops
with no any operations. Therefore, we need a trick which creates
a long-running for loop without being removed by compiler.
Fortunately, we can use `__asm__ __volatile__("" ::: "memory");` to do
the trick.

So you may ask another question: why `__asm__ __volatile__("" ::: "memory");`
can fulfill the task?

According in this post [^2], the `__asm__ __volatile__("" ::: "memory");`
creates a compiler barrier. What's more, with the help of `volatile`
keyword, compiler won't take any optimization of this assembly.
Therefore, we create a statement which doing nothing. Thus, the long-running
for loop serves its purpose. 

## References

[^1] https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#text=rdtsc

[^2] https://preshing.com/20120625/memory-ordering-at-compile-time/
