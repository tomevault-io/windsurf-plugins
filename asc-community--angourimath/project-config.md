---
trigger: always_on
description: For AI agents working on AngouriMath. Humans: [CONTRIBUTING.md](CONTRIBUTING.md) is yours, and
---

# AGENTS.md

For AI agents working on AngouriMath. Humans: [CONTRIBUTING.md](CONTRIBUTING.md) is yours, and
everything below applies to you too.

AngouriMath is a computer algebra system. The thing being built is *mathematics*, and the code is
how it is expressed. Read this as instructions for doing mathematics well, using C# and F#.

## The one rule everything else follows from

**Be a mathematician first.** When mathematical correctness and backward compatibility disagree,
correctness wins. A published API that returns the wrong answer is not an asset to preserve; it is
a bug with users. Say so in the changelog, and change it.

Saying so is not optional, and it has a place: [BREAKING-CHANGES.md](BREAKING-CHANGES.md). Anything
that makes the same input give a different answer goes there before the branch merges, with the old
value, the new one, and why — measured on a build of each, not read off the diff. A user whose code
depended on the wrong answer deserves to find out from us why it moved, rather than from their own
test suite.

The same goes for convention. If mathematicians write it one way and the library writes it another,
the library is wrong — even where the library's way is defensible in isolation. `arcsinh` is not a
thing; the inverse of `sinh` is an *area*, not an arc, so it is `arsinh` (#687). Follow the notation
of the people who use the subject, not the notation that was easiest to parse.

When you have to choose a convention, **check what the other systems answer** — SymPy,
Mathematica, Maxima — and match the mathematics rather than the language you are writing in.
`mod` takes the sign of the divisor because that is what a mathematician means by mod and what
all three of those give; C's `%` truncates, but C's `%` is an operation on machine integers.
Check it, do not reason about it from memory: this exact case was got wrong first time round.

**Consistency is the point.** [#497](https://github.com/asc-community/AngouriMath/issues/497), the
2.0 paper, names inconsistency as the central defect: *"one may find it inconsistent in a lot of
places in API, behaviour, and internal structure of code."* A rule that fires for `sin` but not
`tan`, a limit that works from both sides but not from one, an evaluation that holds precision for
large numbers and drops it for small — each is a bug even when every individual case is defensible.
When you fix something, ask what else is the same shape, and fix that too, or write down why not.

## Not answering is a legitimate answer. Answering wrongly is not.

The most important distinction in this codebase:

| result | means |
|---|---|
| unevaluated (`Limitf`, the original expression back) | "I could not settle this" |
| `NaN` | "**this does not exist**" |
| a value | "this is the value" |

These are three different claims and they are not interchangeable. Returning `NaN` for a limit you
merely failed to compute is a *wrong answer*, not a graceful failure — it tells the caller the limit
does not exist. Returning an unevaluated node is honest.

So, in order of preference: right answer > no answer > slow answer > wrong answer. A wrong answer is
worse than a hang, because a hang is visible.

Say "no answer" by returning **`null`**, not by handing back an unevaluated node of the expression
you were asked about. `Limitf(this, ...)` looks like the honest answer and is in fact a cycle: the
caller evaluates it to compare, evaluating computes the limit, and computing arrives back where it
started. That overflows the stack, which kills the process rather than raising anything catchable.
`null` reads the same to the caller and terminates.

### But "no answer" is the floor, not the target

Read that ordering forwards. *Right answer* comes first, and refusing is what you do when you have
established there is nothing better — not when the right answer looks like more work than you
wanted. Difficulty is not an argument, and neither is "I cannot promise this lands cleanly": the
way to find out whether a fix lands is to write it and measure it, and a failure you measured is a
finding worth having. These *are* reasons to prefer one fix over another — it degrades output
callers depend on, it rests on an assumption that is not true in general, it cannot be validated by
anything you can run. These are not: it touches more files, it might break tests you would then
have to understand, a smaller change exists that suppresses the wrong answer without producing the
right one.

The worked example is [#757](https://github.com/asc-community/AngouriMath/issues/757).
`(x - a)(x + a) <= 0` was answered with an interval whose endpoints are ordered for one sign of `a`
only, and the two candidate fixes were a case split on that sign, or declining to answer a symbolic
coefficient at all. Refusing is not a fix that works — it is a stopgap — and choosing it would have
been choosing the smaller diff over the answer.

**And before building a case analysis, look for the closed form.** That same issue looked like it
needed three branches on the sign of `a`; the answer is the single interval `[-|a|; |a|]`, because
`min(p, q)` is `(p + q - |p - q|)/2` and `max(p, q)` is `(p + q + |p - q|)/2`. One interval, right

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asc-community/AngouriMath](https://github.com/asc-community/AngouriMath) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
