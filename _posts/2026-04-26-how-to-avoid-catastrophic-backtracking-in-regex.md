---
layout: post
title: "How to Avoid Catastrophic Backtracking in Regex"
category: [programming]
tags: [programming, regular_expression, performance_optimization]
---

We nowadays apply regex for simplexity of filtering the texts.
However, regex sometimes causes a lot of time for execution,
and this phenomanon calls catastrophic backtracking.

There are a lot of ways to prevent this, but first let's talk
the root cause.

## What is Catastrophic Backtracking?

Catastrophic backtracking [^2] is a scene that the regex engine meet
an input string which almost matches but ultimately fails. The reason
is that the overlapping quantifiers (e.g., `*` or `+`) lets the engine
tries hundrends of thousands, or even millions, of permutations
before it finally finds a failure match. Eventually, the regex engine
locks up your CPU and crash your application (even CloudFlare met one
in 2019 [^1]).

### What causes catastrophic backtracking?

Taking input string `"aaaaaaaaaaaaaaaaaaaab"` and regex pattern `(a+)+c`.
Because of the nested `+` signs, the engine can group the `a`s in thousands
of different ways: `(aaaa)(a)`, `(aaa)(aa)`, `(a)(a)(a)...`, etc. When it
hits the `b` at the end and realizes there is no `c`, it backtracks and
tries the next permutation.

## Solutions

1. Eliminate Nested Quantifiers: often, writing a tigher, more precise regex
will solve the problem. Taking the following example:
  - Bad: `(a+)+` or `(a*)*`
  - Good: `a+` or `a*`.
2. Use Atomic Groups (`?>...`): an atomic groups tells the regex engine: *"Once
you match the pattern inside this group, throw away all previous backtracking
points. Do not come back here."*
  - Scenario: Matching an integer or decimal `\d+(\.\d+)?`
  - Atomic Fix: `(?>\d+)(\.\d+)?`
3. Use Posessive Quantifiers (`*+`, `++`, `?+`): possessive quantifiers
are essentially shorthand for atomic groups. Imagine the following:
  - Bad: `".*"`
  - Good: `".*+"` (grabs everything, but if the final quote fails, it
fails instantly without backtracking).
4. Be Specific (Use Negated Character Classes): insteading of telling
the engine to match anything (`.*`), tell exactly what it won't match.
5. Make Alternative Mutually Exclusive: when using the OR `|` operator
inside a repeated group, ensure the options won't match the same text.
  - Bad: `(a|a?b)*` (Both `a` and `a?b` can match an "a", creating
overlapping paths.
  - Good: `(a|b)*`
6. Implement Execution Timeout: implement a safety net always secure
your program especially you are facing user-genereated input. 

## Conclusions

Catastrophic Backtracking happens when the input string is almost matches
but eventually fails the match. We can take a lot of counter measurements,
including eliminating nested quantifiers, atomic groups, and
posessive quantifiers, to prevent such situation happens.

Nevertheless, you should always check which methods are the most
suitable on engineering by your constraints.

For the closing, share your experience of countering catastrophic backtracking!

## References

[^1]: https://blog.cloudflare.com/details-of-the-cloudflare-outage-on-july-2-2019/

[^2]: https://www.regular-expressions.info/catastrophic.html

