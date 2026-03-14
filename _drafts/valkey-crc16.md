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
As I have made a contribution to [sse2neon on CRC32]({% link _posts/2024-02-27-optimize-mm-crc32-u8-conversion-in-sse2neon.md %}) and with the fame of Valkey (yes, I always
like fame so that I can use them to earn money!), I decided to
resolve this issue.

## Kind Communities of Valkey

Thanks to the kind comminities and maintainers, I was
assigned to solve this issue. Furthermore, to let maintainers
track the progress, I made the following tasks for possible
solutions:

1. half-byte lookup table
2. Barrett reduction
3. multi-byte lookup table

## OK, What Does Valkey CRC16 Hash Used for?

Valkey utilize CRC16 to hash the key into slots (with modulo 16384),
which is enabled in clustering mode [^2]. 

What's more, provided by the maintainers, the length of key
is at most 20 Bytes, and the maintainers are pretty concerned
the usage of L1 cache (i.e., avoid evicting the data in L1 cache) [^3]

## Get My Hands Dirty

### half-byte lookup table

> View the implementation here: https://github.com/valkey-io/valkey/pull/2300.

### Barrett reduction

> View the implementation here: https://github.com/valkey-io/valkey/pull/2691

### multi-byte lookup table

> View the implementation here: https://github.com/valkey-io/valkey/pull/2790

## Let's benchmark!

To prove any of my solutions is better, we need to benchmark.
There are two benchmark scenarios: single cluster and multi-cluster (3 primary plus
3 secondary nodes).

I list my benchmark environment:

- CPU: Intel(R) Core(TM) i5-3230M CPU @ 2.60GHz
- RAM: 16 GB
- OS: Linux 6.8.0-101-generic

### single cluster

#### benchmark procedures

```

```

#### benchmark result

##### half-byte lookup table

##### Barrett reduction

##### multi-byte lookup table

### multi-cluster

#### benchmark procedures

```
```

#### benchmark result

##### half-byte lookup table

##### Barrett reduction

##### multi-byte lookup table

## Multi-byte Lookup Table Seems Good, But Why It Doesn't Get Accepted?

By the report from reviewers [^5], the multi-byte lookup table solution
does not yield well on their testing environment, or, almost has the same performance compared
to original one.

For no any choices, I decided to end this optimization journey.

## What I Get When the Journey Ends

- Tuning performance just like quality assurance: you
implement a set of methods to prove the original method
acts inferior than your implementations. But sometimes,
it turns in that the original method is the best.
- Ask loud. By my prior experience in big tech and huge scale of open source
projects, you usually have no any ideas how to run and test the project.
Therefore, always asking (with polite) whenever you encounter any troubles
on testing the project. Not only the community helps you, but also
you help the community to recall their memories and to make the project
better (e.g., Valkey then add a clustering mode benchmark pipeline [^6]). 
- Design a test scenario.
- Profiling tools (FlameGraph)

## Special Thanks

I would like to make a great gratitude to Viktor Söderqvist [^4] who
not only gives me a lot of practical advice, but also makes a great
mentoring of Valkey codebase, esepcially about the test scripts.
Moreover, Viktor teached me how to benchmark on the real scene
and the usage of FlameGraph.

Without the help of Viktor, I would not have gone so far, not to
mentioned how Valkey clustering mode.

## References

[^1]: https://github.com/valkey-io/valkey/issues/2031

[^2]: https://valkey.io/topics/cluster-tutorial/

[^3]: https://github.com/valkey-io/valkey/pull/2691#issuecomment-3402032831

[^4]: https://github.com/zuiderkwast

[^5]: https://github.com/valkey-io/valkey/pull/2790#issuecomment-4012948361

[^6]: https://github.com/valkey-io/valkey/pull/3338
