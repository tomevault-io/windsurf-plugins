---
trigger: always_on
description: BezierKit aims to be bug-free. There must never be a case where a public API gives incorrect output.
---

# Claude hints for BezierKit

## High level goals

BezierKit aims to be bug-free. There must never be a case where a public API gives incorrect output.

BezierKit aims to provide public APIs with maximal accuracy, ideally close to floating point machine-level accuracy.

BezierKit's algorithms must be numerically stable and must not ever "explode" in corner cases.

BezierKit aims for API stability. You must never change the public API without being explicitly directed to do so. You may suggest specific changes but the authorization to make every change must come from a human being.

BezierKit aims to maintain its code quality through development processes such as enforcing linting rules. These processes are managed by humans. You may not change these processes without human authorization.

## Performance test guidelines

You must *always* enable all compiler optimizations for performance tests when measuring the CPU time impact of optimizations. Results of performance tests which were run in debug mode provide no signal and must be disregarded. When optimizing for performance the release build CPU execution time is of utmost importance; CPU execution time of debug builds is of no importance at all.

When comparing performance results you must always ensure it is an apples-to-apples comparison. For example when comparing across branches you should verify the tests are measuring the same thing. Run benchmarks sequentially, never in parallel with other CPU-intensive processes; CPU contention from concurrent worktrees or background tasks is a common source of spurious results.

Performance tests should run long enough to provide good signal. XCTest runs several iterations when measuring performance. 100ms per iteration is a good target and allows for good signal even after further optimization.

Code intended for debug asserts must be completely compiled out of release builds. When optimizing code you should ensure that code that evaluates the assertion condition falls entirely within the assertion conditional argument so that it is not evaluated in release builds. If that strategy results in messy code you must use #if DEBUG surrounding the code which supports the assertion check to ensure the compiler cannot retain the code in release builds.

Checks such as precondition(…) which are not compiled out of release builds are not allowed except for public APIs, and those must:
1. represent a previously documented part of the API contract
2. represent irrecoverable errors

## Optimization guidelines

The most significant optimizations stem from choosing the best algorithms. When deciding on an approach you should consider prior work on solving the problem at hand. Once you have a working initial implementation you should search the literature for potential meaningful improvements to the base algorithm. As an example, after implementing Bezier Clipping you should check if further improvements are possible through "Curve intersection using hybrid clipping" by Qi Lou and Ligang Liu.

You must always eliminate the following overheads, which you can locate by inspecting the output assembly of the library compiled with optimizations enabled.
— Heap allocations. Creating Swift arrays for intermediate calculations or return results from internal/private functions almost always results in heap allocations and is undesirable. Some strategies to avoid this include using withUnsafeTemporaryAllocation, invoking a closure on each item (instead of returning them), or using fixed sized struct (eg BernsteinPolynomial1, BernsteinPolynomial2, etc).
— Swift exclusivity access (eg swift_beginAccess). When this overhead appears in profiling results this can sometimes be eliminated using @inline(__always). You must always run performance tests with and without the hint and confirm a substantial speedup before keeping it. @inline(__always) can cause code-size bloat that hurts unrelated call sites, so it must never be used speculatively — only when benchmarks confirm a clear improvement.
— Protocol witness table dispatch. Avoid existentials (`any Protocol`) in hot paths; use generics with concrete type constraints instead, which allows the compiler to specialize and inline.

You should use FMA (fused multiply-add) instructions when they speed execution without accuracy loss. For example Swift Float's addingProduct has been successfully applied to speed execution without decreasing accuracy.

When optimizing you must never trade implementation accuracy for speed. It is an error to decrease the order of magnitude of accuracy of a public API and you should verify the accuracy of the new approach. When you provide a new implementation you must compare the accuracy against the old one on test data.

When optimizing you must always consider the numerical stability of new implementations. For example, if a change introduces a division by an argument you must consider if the solution will "explode" when the denominator of the division (the argument) is nearly zero. It is an error to provide a new implementation whose accuracy degrades the accuracy of public APIs on worst case input examples.

## Code style

Code readability is important

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hfutrell/BezierKit](https://github.com/hfutrell/BezierKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
