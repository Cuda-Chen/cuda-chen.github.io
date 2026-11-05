---
layout: post
title: "Insert Multiple Characters at Once: simdjson prettification Optimization"
category: [Programming]
tags: [Open Source Contribution, C++, JSON, simdjson]
---

Indentation is a vital work for human beings as it
can reduce the stress of reading and reviewing the
code and document. Moreover, it provides a distinct
message to show whether a text section has any connections
to other sections or not. Many tools such as clang-format [^2]
are proposed to have the enrichment of indentation to fulfill the user needs,
providing with speed of indentation.

There is no exception of JSON [^1] that indentation helps
us to read JSON with less pressure. The most intuitive
ways to do so is to insert the character for indentation
one-by-one just like my previous PR on simdjson [^3], which
yields the sub-optimal performance. As a result, I am going
to show how to insert multiple characters at once, yielding
better performance.

## Test Environment

This test environment of my contribution depicts as follows:

- OS: Linux Kernel 6.8.0
- CPU: Intel(R) Core(TM) i5-3230M CPU @ 2.60GHz
- RAM: 16 GB
- Compiler: GCC 13.3.0

For the Document/sec it is a customized measurement unit
that shows how many documents can be processed per second.

I utilize the Google benchmark [^5] as the benchmark framework,
and choose the famous twitter.json [^6] as the benchmark input.

## Inserting Multiple Characters at Once Performtly

For the naive method, you can create a string filled with
indentation character and append in the beginning of each
line (remember the depth of each code block, of course).
Yet, this creates two problems: you need to create the indentation
string with current depth times the count of indentation
character.

Let's take another approach: we can pre-allocate a sufficiently
large length of the indentation string. When we need to
insert indent, we just return the `string_view` (with desired
length) of the indentation string. For example, my first
version of the PR [^4]:

```
diff --git a/include/simdjson/dom/serialization-inl.h b/include/simdjson/dom/serialization-inl.h
index 74e4896b6c..f540e53544 100644
--- a/include/simdjson/dom/serialization-inl.h
+++ b/include/simdjson/dom/serialization-inl.h
@@ -377,12 +377,14 @@ simdjson_inline void mini_formatter::print_space() { return; }
 simdjson_inline void pretty_formatter::print_newline() { one_char('\n'); }

 simdjson_inline void pretty_formatter::print_indents(size_t depth) {
-  if (this->indent_step <= 0) {
-    return;
-  }
-  for (size_t i = 0; i < this->indent_step * depth; i++) {
-    one_char(' ');
-  }
+  if (indent_step <= 0) return;
+  static constexpr char SPACES[] =
+      "                                                                "
+      "                                                                "
+      "                                                                "
+      "                                                                ";
+  size_t total_spaces = (std::min)(depth * static_cast<size_t>(indent_step), sizeof(SPACES) - 1);
+  chars(SPACES, SPACES + total_spaces);
 }

 simdjson_inline void pretty_formatter::print_space() { one_char(' '); }
```

For the performance it can reach for about 36% more:

```
Metric         │ master branch  │ this PR        │ Performance Gain        
───────────────┼────────────────┼────────────────┼─────────────────────────
CPU Time       │ 9,818,340 ns   │ 7,202,667 ns   │ 1.36x faster (-26.6%)   
Throughput     │ 78.15 MB/s     │ 106.53 MB/s    │ +36.3%                  
Documents/sec  │ 101.85 /s      │ 138.84 /s      │ +36.3%   
```

## Pushing Further

Suggested by mvanslobbe [^7], we can use the existing
`grow()` function in `struct vector_with_small_buffer` [^8]
to allocate the indentation string efficiently by using
`std::copy()`, thus no need
to store the pre-allocated indentation string.

For the performance, it further reached for about 37% more:

```
Metric         │ master branch  │ this PR        │ Performance Gain        
───────────────┼────────────────┼────────────────┼─────────────────────────
CPU Time       │ 9,818,340 ns   │ 7,120,913 ns   │ 1.38x faster (-27.5%)   
Throughput     │ 78.15 MB/s     │ 107.75 MB/s    │ +37.9%
Documents/sec  │ 101.85 /s      │ 140.43 /s      │ +37.9%                  
```

Furthermore, it gains about 33% more on a 40-level deep nesting JSON:

```
Metric         │ master branch  │ this PR        │ Performance Gain        
───────────────┼────────────────┼────────────────┼─────────────────────────
CPU Time       │ 64,548 ns      │ 48,377 ns      │ 1.33x faster (-25.1%)   
Throughput     │ 101.66 MB/s    │ 135.64 MB/s    │ +33.4%
Documents/sec  │ 15.49k /s      │ 20.67k /s      │ +33.4%                  
```

## Recap

In this post, I first illustrate what the indentation is with its
application for human readability and performance, especially on JSON. 
Next, I show the progress of my indentation implementation on simdjson
with benchmarking, leading the indentation to at least 1.37x speedup
compared to baseline.

As inserting string is still a tough question today, feel free
to share your experience of your string insertion journel with
comments!

## Special Thanks

I would like to give specials thank to mvanslobbe [^9] for assisting
me about the optimization ideas and point out the flaws of
the indentation depth count. What's more, mvanslobbe also gives
me some ideas about benchmarking on CITM and 40-level nesting,
making the benchmark has the actual meaning when facing to
real world usage.

Also, I would like to give special thanks to lemire [^10],
the maintainer of simdjson. lemire not only reviews my changes
with pratical suggestions, but also helps me out of the code
merge.
 
## References

[^1]: https://www.json.org/json-en.html

[^2]: https://clang.llvm.org/docs/ClangFormat.html

[^3]: https://github.com/simdjson/simdjson/pull/2026

[^4]: https://github.com/simdjson/simdjson/pull/2916

[^5]: https://github.com/google/benchmark

[^6]: https://github.com/simdjson/simdjson/blob/master/jsonexamples/twitter.json

[^7]: https://github.com/simdjson/simdjson/pull/2916

[^8]: https://github.com/Cuda-Chen/simdjson/blob/91fa0ff5cd378ba59d97a027e6c6f6577cc28b2a/include/simdjson/dom/serialization.h#L121

[^9]: https://github.com/mvanslobbe

[^10]: https://github.com/lemire
