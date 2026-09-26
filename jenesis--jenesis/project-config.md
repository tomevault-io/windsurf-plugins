---
trigger: always_on
description: Jenesis, the build tool, and jpx, the module runner, in one repository. The tool builds itself: `build/jenesis`
---

# AGENTS.md

Jenesis, the build tool, and jpx, the module runner, in one repository. The tool builds itself: `build/jenesis`
links to `sources/build/jenesis`, so `java build/jenesis/Make.java` compiles, tests and packages this
project with the very sources it is working on. `README.md` covers the layout, the demos, CI and releasing;
this file is how the code is written and changed. The user documentation is
[jenesis.build](https://jenesis.build) ([jenesis/jenesis-documentation](https://github.com/jenesis/jenesis-documentation)),
and `java build/jenesis/Make.java skill` prints an onboarding briefing from the same material.

## Build & test

- **JDK 25 or newer**, nothing else. `java build/jenesis/Make.java` builds and tests everything;
  `stage` lays out the release tree; `pin` rewrites the pins; `help` lists every selector and `-D` flag.
- One test class: `java -Djenesis.test.filter='.*BuildExecutorTest' -Djenesis.test.force=true
  -Djenesis.print.tests=true build/jenesis/Make.java +tests`. The filter is a regex over class names,
  `force=true` runs tests whose inputs did not change, `print.tests` streams the JUnit output. Classes named
  `*RunTest` run the real external tool and need the network.
- CI builds under `-Djenesis.dependency.pin=strict`: after changing a dependency, run `pin` and commit the
  rewritten `module-info.java` / `pom.xml` lines, or CI fails on the unpinned coordinate.
- When the tool cannot build itself, `mvn test` (the root `pom.xml`) compiles and tests the sources without
  Jenesis. It validates only; it stages and pins nothing.
- Every feature ships a demo under `demo/` with a README and a verification command in
  `.github/workflows/demos.yml`, which `build.yml` calls once per operating system; the demos are the
  end-to-end suite and the documentation's examples.

## How the code is written

**No comments, no Javadoc.** The sources carry neither, and a change adds none - not inline, not on a type,
not on a method, in `sources/` or `tests/`. A name, a type or a smaller method carries the meaning; if a
construct seems to need a comment, restructure it or name it better. The one place Javadoc appears is
`module-info.java`, because the `@jenesis.*` tags there (`@jenesis.main`, `@jenesis.test`, `@jenesis.pin`,
…) are configuration the tool reads, not commentary.

**One caller, no method.** A private method with one caller is inlined at it. It survives only where
inlining would hurt: a self-contained algorithm that has a name, a body with early returns that the caller
would have to be restructured around, one holding an anonymous class, or a pattern switch over a sealed type.
A static whose first argument is one of our own types belongs on that type as an instance method instead -
without inventing a type to make the call virtual.

**Static fields are one block.** No blank line separates them, and constants that belong together share one
declaration, wrapped onto continuation lines where they are long
(`private static final String MAVEN_GROUP = "org.antlr", MAVEN_ARTIFACT = "antlr4";`). A separate declaration
is what marks a separate concept, so a `Set` or `Pattern` initializer keeps its own line, where a declarator
comma would read as one of its own.

**Zero dependencies.** The tool ships as source, vendored into every project that uses it, and runs with a
JDK and nothing else - that is the promise, and it is not negotiable. `build.jenesis` `requires` only
`jdk.compiler` and `java.xml`; there is no third-party library anywhere in `sources/`, and none is added for
convenience. What the JDK does not provide is written against what it does: JSON is `Json.java` over
`java.base`, HTTP is `HttpURLConnection`, XML is `java.xml`. A tool the build needs at run time - a compiler, a
linter, a packager - is resolved as a dependency in its own group and forked or loaded in a module layer,
never added to the module's own `requires`.

**Java 25 idiom.** Every file starts with `import module java.base;` (plus `jdk.compiler` or `java.xml` where
used). Records, sealed types, pattern switches and unnamed variables (`_`) are the normal idiom.

**Immutable records with withers.** Configuration objects are records or final classes whose state never
changes after construction. Each exposes one method per component, named exactly like the component, that
returns a new instance with that value replaced (`new Project(Path.of(".")).version("1.0.0").sources(true)`,
`new BuildExecutor.Configuration().concurrency(4)`). No setters, no builders, no `with` prefix.

**System properties are the defaults.** Every setting is a `jenesis.<area>.<name>` system property, read
once where the object is built:

- `ofEnvironment(environment, …)` is the static factory that reads every setting from the `Environment` it is
  given (`environment.value("executor.digest", "MD5")`, `environment.number("executor.concurrency", 0)`,
  `environment.flag("source.pmd", true)`), with a value the object cannot do without following it as a
  further argument (`Project.ofEnvironment(environment, root)`);
- the public no-argument or short constructor reads nothing and takes each setting's default, exactly as

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jenesis/jenesis](https://github.com/jenesis/jenesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
