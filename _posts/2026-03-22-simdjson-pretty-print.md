---
layout: post
title: "simdjson Contribution: Implement Pretty Print of DOM"
category: [open source contribution, programming, SIMD]
tags: [JSON, C++, SIMD, prigramming]
---

## Preface

Years ago, I was finding some projects related to SIMD for
 taking a break after making contributions to sse2neon.
At the same time, I was working in a big tech company, and
one of my colleague, a senior C++ developer, said the magic
of simdjson, which can parse gigabytes of JSON per second [^1].

In order to make some remarks in this world, I started to find
whether there are any issues that I can take to sharpen my skills
on SIMD and C++. Luckily, I spotted this issue: "Pretty print for DOM" [^2],
and I thought it was a great starting point.

## A Brief Introduction to simdjson

simdjson is a C++ based library that parses gigabytes of JSON per second.
It also provides a strict validation on JSON and UTF-8, and it applies
a two stage algorithm which utilizes SIMD instructions [^3]. Because of
its superior performance, it is widely adopted on many projects including
the Node.js runtime.

## What I'm Going to Do

- Create a indented JSON string.
- Performance priority #1.

## The Evolution of My Implementation

At first, I implemented in an inheritance way:

```cpp
class mini_formatter {
  mini_formatter() = default;
  virtual ~mini_formatter() = default;
};

class pretty_formatter : public mini_formatter {
  pretty_formatter() {}
  ~pretty_formatter() {}
};
```

Suggested by a reviewer [^4], I altered my implementation
into a CRTP [^5] solution:

```cpp
template<class formatter>
class base_formatter {  
  simdjson_inline void call_print_newline() {
      this->print_newline();
  }

  simdjson_inline void call_print_indents(size_t depth) {
      this->print_indents(depth);
  }

  simdjson_inline void call_print_space() {
      this->print_space();
  }
};

class mini_formatter : public base_formatter<mini_formatter> {
public:
  simdjson_inline void print_newline();

  simdjson_inline void print_indents(size_t depth);

  simdjson_inline void print_space();
};

class pretty_formatter : public base_formatter<pretty_formatter> {
public:
  simdjson_inline void print_newline();

  simdjson_inline void print_indents(size_t depth);

  simdjson_inline void print_space();

protected:
  int indent_step = 4;
};
```

After applying CRTP, my implementation achieved speed improvement up to 10x
(from 2s to 0.2s) on the testing dataset.

For the entire PR, you can visit here: https://github.com/simdjson/simdjson/pull/2026.

## Conclusion and Future Aspect

In this PR, I not only implemented the JSON indentation feature, but
also proved that C++ templating can enhance performance.

For the future aspect, I am going to add the indentation spaces feature (i.e.,
user can set 3 space indentation).

## References

[^1]: https://github.com/simdjson/simdjson

[^2]: https://github.com/simdjson/simdjson/issues/1329

[^3]: https://branchfree.org/2019/02/25/paper-parsing-gigabytes-of-json-per-second/

[^4]: https://github.com/simdjson/simdjson/pull/2026#discussion_r1252196548

[^5]: https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern
