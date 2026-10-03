---
trigger: always_on
description: ergo is a Rego library that turns policy evaluation into a structured report. Users copy `ergo.rego` into their own projects, so it has to stay a single file with no dependencies.
---

# Working on ergo

ergo is a Rego library that turns policy evaluation into a structured report. Users copy `ergo.rego` into their own projects, so it has to stay a single file with no dependencies.

## Files

- `ergo.rego` is the whole library.
- `ergo_test.rego` holds its tests.
- `custom_op_test.rego` defines custom operators that only the tests use.
- `README.md` walks a new user through a first policy.
- `REFERENCE.md` describes every field, operator, cause and report entry.
- `examples/` holds worked examples. Each one has an input, the same policy in plain Rego and with ergo, and tests that pin what both versions report.

## Checks

Run these before saying a change is done:

```sh
opa check --strict . --ignore .github
opa fmt --list .
opa test . --ignore .github
opa eval --strict-builtin-errors --ignore .github -d . --format raw 'count(data) > 0'
regal lint --disable-all --enable-category bugs --disable redundant-existence-check --ignore-files '.github/**' .
```

`opa fmt --list .` should print nothing. If it prints file names, run `opa fmt -w .`.

The `opa eval` command runs every test again with `--strict-builtin-errors`, which turns a built-in given the wrong type into an error, and should print `true`. If it prints an error, check the value's type before the built-in on that line, so that ergo gives the same report with or without the flag.

The `regal lint` command runs [Regal](https://www.openpolicyagent.org/projects/regal)'s rules for likely bugs, and should say `No violations found`. One of them, `leaked-internal-reference`, fails when a file outside `package ergo` calls a rule whose name starts with `_`. Test files are allowed to. `redundant-existence-check` is off because it flags `_verdict_cause(passed) := "satisfied" if passed`, where `if passed` is what stops a `false` from counting as satisfied. The flags are on the command line and not in `.regal/config.yaml` because OPA would load that file as data.

OPA loads every JSON and YAML file it finds as data. The workflow files under `.github` clash with each other, so the checks ignore that folder.

CI runs these checks on pull requests and on pushes to `main` (`.github/workflows/test.yml`), using the OPA version the README names and Regal 0.43.0. When you change the OPA version, change it in the README, in every job of the workflow and in the Docker command below.

CI also runs every test compiled to Wasm, with `opa test . --ignore .github --target wasm`, because Wasm walks objects and sets in a different order from `opa eval`, so anything that ends up in the report has to be sorted. That needs the Linux build of OPA. The macOS one says `engine not found`, so on a Mac run it in Docker:

```sh
docker run --rm -v "$PWD":/src:ro -w /src openpolicyagent/opa:1.19.0 test . --ignore .github --target wasm
```

CI also runs every test with OPA's JavaScript runtime, `@open-policy-agent/opa-wasm` (`.github/wasm-js.cjs`), because that runtime brings its own versions of some built-ins, like a `sprintf` that formats lists and objects differently, and lacks others. The script compiles each test as a Wasm entrypoint, fails if ergo uses a built-in the runtime doesn't have (apart from `time.parse_rfc3339_ns`, which it passes in, as `REFERENCE.md` tells Wasm users to), and then runs every test. So pass `sprintf` only strings: write anything else with `_text`.

CI also fails when a line of Rego isn't reached by any test. To list those lines yourself:

```sh
opa test . --ignore .github --coverage | jq -r '.files | to_entries[] | .key as $f | .value.not_covered[]? | "\($f):\(.start.row)"' | sort -u
```

## Tests

Every change needs tests: new behaviour, bug fixes and changes to existing behaviour alike. A bug fix starts with a test that fails because of the bug. A change without tests isn't finished.

## No comments

There are no comments in the code or the tests, and it should stay that way.

The one exception is the license header at the top of `ergo.rego`. Users copy that file on its own, so the header keeps the license with it.

CI fails on any other comment in a `.rego` file. It asks `opa parse` for each file's comments, so a `#` inside a string doesn't count. To list them yourself:

```sh
git ls-files '*.rego' | while read -r f; do opa parse "$f" --format json --json-include comments,locations | jq -r --arg f "$f" '.comments[]? | .Location.row as $r | select(($f == "ergo.rego" and $r <= 2) | not) | "\($f):\($r)"'; done
```

When something in the code looks like it could be simplified but mustn't be, a test says so instead of a comment. Give the test a name that explains the reason, like `test_out_of_scope_subject_is_recorded_as_evidence`. If you're about to write a comment, write a test.

Before removing or simplifying code, run the tests. Many of them exist to stop "obvious" simplifications that would let a check pass when it should fail.

## Failing closed

When ergo can't read something or isn't sure, the check fails. It never passes by accident and never disappears from the report. Every change must keep this true, and every new operator needs tests for missing, `null` and wrong-typed input.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kosli-dev/ergo](https://github.com/kosli-dev/ergo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
