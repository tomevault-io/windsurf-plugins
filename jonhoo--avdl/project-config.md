---
trigger: always_on
description: A Rust implementation of Apache Avro's IDL (`.avdl`) compiler, porting
---

# avdl — Avro IDL Compiler in Rust

A Rust implementation of Apache Avro's IDL (`.avdl`) compiler, porting
the Java `avro-tools idl` and `avro-tools idl2schemata` subcommands.
Parses `.avdl` files using an ANTLR4-generated parser and emits Avro
Protocol JSON (`.avpr`) or Schema JSON (`.avsc`).

## Specification references

- [Avro specification](https://avro.apache.org/docs/1.12.0/specification/)
  — full schema, protocol, and serialization format.
  Local source: `avro/doc/content/en/docs/1.12.0/Specification/_index.md`
- [Avro IDL language](https://avro.apache.org/docs/1.12.0/idl-language/)
  — the `.avdl` surface syntax this tool parses.
  Local source: `avro/doc/content/en/docs/1.12.0/IDL Language/_index.md`

These are the authoritative Hugo source files for the published docs
at avro.apache.org. Prefer reading the local copies over fetching the
website — they are always available and match the submodule version.

## Build and test

```sh
cargo build
cargo test                              # unit tests + integration tests
cargo insta test --unreferenced delete  # also cleans up orphaned .snap files
```

The integration tests parse `.avdl` files from the Avro test suite and
compare the serialized JSON output against golden `.avpr`/`.avsc` files.
Known issues are tracked in `issues/`.

**Non-goal: byte-identical output.** Producing output that is
byte-for-byte identical to the Java tool is explicitly a non-goal.
Whitespace differences (spaces before colons, array/object
line-breaking style, indentation) are expected and acceptable. The
goal is **semantic correctness**: the JSON output should parse to the
same logical structure as the Java tool's output. Always compare
semantically (parse both as JSON and compare values) rather than as
raw strings.

### Comparing against avro-tools

```sh
INPUT_DIR=avro/lang/java/idl/src/test/idl/input
OUTPUT_DIR=avro/lang/java/idl/src/test/idl/output
CLASSPATH_DIR=avro/lang/java/idl/src/test/idl/putOnClassPath

# Rust:
cargo run -- idl --import-dir $INPUT_DIR --import-dir $CLASSPATH_DIR \
  $INPUT_DIR/foo.avdl tmp/foo.avpr
# Java:
java -jar ../avro-tools-1.12.1.jar idl $INPUT_DIR/foo.avdl tmp/foo-java.avpr
# Compare (semantic):
diff <(jq -S . tmp/foo.avpr) <(jq -S . $OUTPUT_DIR/foo.avpr)
```

The canonical correctness check is `cargo test`, which runs semantic
JSON comparison against golden files for both `idl` (18 `.avpr` files)
and `idl2schemata` (61 `.avsc` files across 17 test inputs).

For ad-hoc debugging, create a temporary Rust example in `examples/`
and run it with `cargo run --example <name>`. Remove the example
after use.

Use `tmp/` (project-local) for intermediate files and comparison
artifacts, not `/tmp`. This keeps outputs discoverable and
project-scoped. The `tmp/` directory is gitignored.

### Helper scripts

`scripts/compare-adhoc.sh` compares Rust output against Java
avro-tools output for arbitrary `.avdl` files written to `tmp/`.
Use this for edge-case exploration beyond the golden test suite.

```sh
scripts/compare-adhoc.sh tmp/edge-case.avdl          # single file
scripts/compare-adhoc.sh tmp/edge-*.avdl              # glob multiple files
scripts/compare-adhoc.sh --idl2schemata tmp/test.avdl # idl2schemata mode
scripts/compare-adhoc.sh --show-output tmp/test.avdl  # print JSON on diff
scripts/compare-adhoc.sh --rust-only tmp/test.avdl    # skip Java comparison
scripts/compare-adhoc.sh --import-dir path/ tmp/t.avdl # with import dirs
```

### Ad-hoc testing with the CLI

When testing the CLI with ad-hoc `.avdl` input, **write the input to a
temp file in `tmp/`** and pass it by path, rather than piping via
`echo | cargo run` or `cat <<EOF | cargo run`. This avoids interactive
permission prompts for pipe commands in sub-agents.

```sh
# Good: write to temp file, pass by path
cat > tmp/test-$(uuidgen).avdl <<'EOF'
protocol Test { record Foo { string name; } }
EOF
cargo run -- idl tmp/test-*.avdl

# Avoid: piping requires interactive permission
echo 'protocol Test { ... }' | cargo run -- idl
```

### Sandbox pipe workaround

The Claude Code sandbox has a [known issue][cc-16305] where data is
silently dropped in shell pipes between commands. Appending a trailing
`;` to the command fixes this:

```sh
# Broken in sandbox (downstream receives no input):
diff <(jq -S . a.json) <(jq -S . b.json)

# Fixed — append `;`:
diff <(jq -S . a.json) <(jq -S . b.json);
echo "abc" | grep "abc";
```

This affects pipes (`|`), process substitution (`<(...)`), and any
command that connects stdout of one process to stdin of another.
Direct command invocations without piping (e.g., `cargo run`,
`java -jar`) work fine under sandbox.

[cc-16305]: https://github.com/anthropics/claude-code/issues/16305

### Sandbox `!` (negation) workaround

The Claude Code sandbox has a [separate bug][cc-24136] where the bash
`!` keyword (pipeline negation operator) is treated as a literal
command name instead of a shell reserved word. The command after `!`
**never executes** — no side effects occur, and stderr shows
`!: command not found`. This affects `if !`, `while !`, and bare `!`.

The trailing-semicolon workaround for the pipe bug above does **not**
fix this.

```sh
# Broken in sandbox (touch never runs):
if ! some_command; then handle_failure; fi


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonhoo/avdl](https://github.com/jonhoo/avdl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
