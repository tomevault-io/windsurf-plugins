---
trigger: always_on
description: Guidance for AI-assisted work in this repository. This is a security-critical library. When in doubt, do less and ask a maintainer.
---

# AGENTS.md

Guidance for AI-assisted work in this repository. This is a security-critical library. When in doubt, do less and ask a maintainer.

## Project

OWASP Java HTML Sanitizer (`com.googlecode.owasp-java-html-sanitizer:owasp-java-html-sanitizer`). Takes attacker-controlled HTML and produces HTML that is safe to embed in a web page. Allowlist policies are built with `HtmlPolicyBuilder` and compiled into thread-safe `PolicyFactory` instances.

This is a multi-module Maven build:

- `owasp-java-html-sanitizer/` — the library. Core code: `owasp-java-html-sanitizer/src/main/java/org/owasp/html/`. Tests: `owasp-java-html-sanitizer/src/test/java/org/owasp/html/`. `src/main/java9/module-info.java` is the JPMS module descriptor. A second compiler execution compiles it for Java 9 into `META-INF/versions/9/` of the multi-release JAR, which is how a Java 8 JAR carries one; it exports `org.owasp.html` only. `src/it/jpms-consumer/` is a small module that `./mvnw verify` compiles and runs against the packaged JAR to check that the descriptor is read and the shim stays encapsulated. Those two are the only Java 9 sources in the library module.
- `java8-shim/`, `java10-shim/` — in-repo shims that let the Java 8 library use newer JDK collection APIs when running on Java 10+. Both are inlined into the library JAR by the shade plugin and do not appear in the published POM. `java10-shim` is the only module that may use Java 9 and 10 APIs (`maven.compiler.release` 10). Nothing uses anything newer.
- `empiricism/` — in-browser experiments that record how real browsers nest and balance tags. They generate `HtmlElementTablesCanned.java` in the library's core package. Never hand-edit that file: change the probe or `empiricism/canned-data.json` and run `empiricism/rebuild.sh`.
- `examples/`, `docs/` — samples and documentation. Not part of the shipped artifact.

Primary threat: XSS, including mutation XSS (mXSS), namespace confusion, and browser parser quirks.

## Build and Test

- Build and run all tests: `./mvnw clean verify` (or `mvn clean verify`). This is what CI runs.
- Tests only: `./mvnw test`
- Always build from the repo root across the whole reactor. Building the library module alone (`-pl owasp-java-html-sanitizer`) fails to resolve the shim modules unless they were installed first; add `-am` if you must narrow the build.
- Building requires JDK 11 or newer (enforced by the Maven Enforcer plugin); CI runs on JDK 11, 17, 21, and 25. Prefer `./mvnw`, which pins a Maven version that satisfies the enforcer.
- The library targets Java 8 source and bytecode (`maven.compiler.release` 8) regardless of the JDK used to build, and test code is compiled at the same level. Do not use newer language features or APIs in either. The module descriptor and the modular consumer check are the exceptions, compiled for Java 9 and nothing newer.
- Tests use JUnit 5 (Jupiter). Do not add JUnit 4 style tests.
- Fuzzer tests (`*FuzzerTest.java`) and the AntiSamy test suite are part of the normal test run. The fuzzers seed from the clock and report the seed on failure; reproduce with `-Djunit.seed=<seed>`. A fuzzer failure is a real bug, never something to re-run until it passes.
- All tests must pass before proposing any change. No exceptions.

## Security Invariants

Hard rules. Never violate them, never refactor around them.

1. **Allowlist only.** Everything is denied unless a policy explicitly allows it. Never introduce denylist or strip-known-bad logic.
2. **Default deny for anything new.** New elements, attributes, URL protocols, or CSS properties are unsafe until a policy author explicitly allows them.
3. **Output must be safe in every HTML parsing context, not just the happy path:** foreign content (SVG, MathML), raw-text and RCDATA elements (`script`, `style`, `title`, `textarea`, `noscript`, `xmp`, `iframe`, `plaintext`), comments, CDATA. `HtmlTextEscapingMode` is the sanitizer's table of these. Browsers treat `noscript` as raw text when scripting is on even though the sanitizer deliberately does not; CVE-2025-66021 exploited that mismatch.
4. **Case sensitivity matters.** SVG and MathML element and attribute names are not lower-cased (see the mixed-case foreign name sets in `HtmlLexer`); SVG `textArea` is distinct from HTML `textarea`. Preserve this.
5. **Encoding and rendering are part of the security boundary.** Changes to `Encoding`, `HtmlStreamRenderer`, `TagBalancingHtmlStreamEventReceiver`, `HtmlTextEscapingMode`, entity handling, or serialization are security changes and require adversarial tests.
6. **The prepackaged policies in `Sanitizers` are published safety promises.** Never widen what they allow.
7. **Round-trip safety:** sanitized output, re-parsed by a browser, must not mutate into a different, unsafe DOM (mXSS).

## Change Rules

- Small, single-purpose changes only. No drive-by refactors of parser, policy, or renderer code.
- Any change touching parsing, policy enforcement, rendering, encoding, URL handling, or CSS handling requires new regression tests with hostile payloads, not just benign inputs.
- Never weaken, delete, or loosen an existing test to make a change pass. A failing security test means the change is wrong.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OWASP/java-html-sanitizer](https://github.com/OWASP/java-html-sanitizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
