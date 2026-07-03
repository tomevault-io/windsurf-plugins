---
trigger: always_on
description: This document contains information for Claude Code (or other AI assistants) working on the sloth project.
---

# Claude Code Development Guide

This document contains information for Claude Code (or other AI assistants) working on the sloth project.

## Project Structure

- **core/** - Core bytecode analysis and transformation logic (published; `-release 9`)
- **agent/** - Java agent that patches lazy vals at class-load time (published, shaded fat jar; `-release 9`)
- **cli/** - Command-line interface for running transformations (`-release 9`)
- **testops/** - Development tooling + shared test infra (`ExampleLoader`, `ExampleRunner`, `TestPaths`)
- **tests/** - Test fixtures only (`src/test/resources/fixtures/examples/`); no sbt module
- **tests-jdk11/** - Test suites that run on Java 11 (analysis + Java-9 runtime/classfile-version proof)
- **tests-jdk25/** - Test suites that run on Java 24+ (sun.misc.Unsafe warning assertions)

## Development Tools

### Compiling Test Examples

The `compileExamples` command compiles all test fixtures across multiple Scala versions and generates javap outputs for bytecode inspection.

**Usage:**

```bash
# Compile examples without patching
sbt compileExamples

# Compile examples and generate patched versions (3.3-3.7)
sbt compileExamplesWithPatching

# Run with example filtering
SELECT_EXAMPLE=simple-lazy-val sbt compileExamples
SELECT_EXAMPLE=simple-lazy-val,class-lazy-val sbt compileExamplesWithPatching

# Or run the assembly directly
sbt testops/assembly
java -jar testops/target/scala-3.3.8/sloth-testops.jar
java -jar testops/target/scala-3.3.8/sloth-testops.jar --patch
```

**What it does:**

1. Discovers all examples in `tests/src/test/resources/fixtures/examples/`
2. Compiles each example with all test Scala versions:
   - 3.0.2, 3.1.3, 3.2.2
   - 3.3.0, 3.3.6, 3.4.3, 3.5.2, 3.6.4, 3.7.3
   - 3.8.1
3. Generates javap disassembly (`.javap.txt`) for each compiled classfile
4. With `--patch` flag: Transforms Scala 3.3-3.7 classfiles to use VarHandle-based lazy vals (like 3.8+)
5. Outputs everything to `.out/` directory

**Output structure:**

```
.out/
  <example-name>/
    <scala-version>/
      *.class           # Compiled classfiles
      *.javap.txt       # Javap disassembly
      *.scala          # Source files (copied)
      .scala-build/    # scala-cli build artifacts
    patched/           # Only present when using --patch flag
      3.3.0/           # Patched versions (3.3-3.7 only)
        *.class        # Patched classfiles with VarHandle-based lazy vals
        *.javap.txt    # Javap disassembly of patched files
      3.3.6/
      3.4.3/
      3.5.2/
      3.6.4/
      3.7.3/
```

**Inspecting results:**

```bash
# List all examples
ls .out/

# View javap output for a specific version
cat .out/companion-object-lazy-val/3.3.0/Foo$.javap.txt

# View patched javap output
cat .out/companion-object-lazy-val/patched/3.3.0/Foo$.javap.txt

# Compare lazy val implementations across versions
grep -h "OFFSET\|bitmap\|lzyHandle" .out/simple-lazy-val/*/SimpleLazyVal$.javap.txt

# Compare original vs patched (OFFSET -> VarHandle)
diff .out/simple-lazy-val/3.3.0/SimpleLazyVal$.javap.txt .out/simple-lazy-val/patched/3.3.0/SimpleLazyVal$.javap.txt

# Count generated files
find .out -name "*.javap.txt" | wc -l
```

**Use cases:**

- Debugging lazy val detection across Scala versions
- Comparing bytecode patterns between versions
- Verifying transformation correctness
- Understanding lazy val implementation changes
- Testing bytecode patching by comparing original vs patched implementations
- Inspecting VarHandle vs Unsafe-based lazy val bytecode

## Testing

### Tests are split by required JVM — `sbt test` is disabled

Scala 3.8 requires JDK 17 and can't emit bytecode below v61, so this build uses **Scala 3.3.8** (LTS,
Java-8 stdlib) with `-Yfuture-lazy-vals` (keeps the Unsafe-free VarHandle lazy-val scheme) and
`-release 9` on the published modules (`core`, `agent`, `cli`). Artifacts are therefore loadable on
Java 9. The test suites are split into two sbt modules by the JVM they need, and a plain `sbt test`
is intentionally disabled (it errors with a pointer to the two targets):

- **`sbt tests-jdk11`** (module `tests-jdk11`, run on **Java 11**) — pure bytecode-analysis suites
  (`LazyValDetectionTests`, `SemanticLazyValComparisonTests`, `AgentPatchingTests`), the Java-9
  runtime proof (`Jdk9RuntimeTests` runs the agent + VarHandle-patched 3.3–3.7 code), and
  `ClassfileVersionTests` (asserts the agent jar + core are ≤ v53). Locally:
  `JAVA_HOME=~/.sdkman/candidates/java/11.0.31-tem`.
- **`sbt tests-jdk25`** (module `tests-jdk25`, run on **Java 24+**) — `BytecodePatchingTests` and
  `AgentIntegrationTests`, which assert presence/absence of the `sun.misc.Unsafe` warning that only
  newer JDKs emit. Locally: `JAVA_HOME=~/.sdkman/candidates/java/25-graalce`.

CI runs these as two jobs (`test-jdk11` on temurin 11, `test-jdk25` on temurin 25). The harness
itself needs Java 11+ (sbt, scala-cli, and testops' jsoniter dependency are >v53), so the actual
Java-9 floor is guaranteed by `ClassfileVersionTests` (published classes ≤ v53), not by executing
on Java 9. Java 9/10 are best-effort.

**IMPORTANT: still narrow with `SELECT_EXAMPLE` / `ONLY_SCALA_VERSIONS`.** A full module run compiles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VirtusLab/sloth](https://github.com/VirtusLab/sloth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
