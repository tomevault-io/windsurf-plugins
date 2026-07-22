---
trigger: always_on
description: rules_python is a Bazel based project. Build and run tests as done in a Bazel
---

# Guidance for AI Agents

rules_python is a Bazel based project. Build and run tests as done in a Bazel
project.

Act as an expert in Bazel, rules_python, Starlark, and Python.

DO NOT `git commit` or `git push` unless given explicit permission.

## RULES TO ALWAYS FOLLOW AND NEVER IGNORE

ALWAYS FOLLOW THESE RULES. NEVER VIOLATE THEM.

Ask for user input and provide a justificaiton if trying to violate them.

* NEVER run `bazel clean --expunge`.
* Once a PR is created, do not amend or rebase.
* Do not add Bazel copyright to new or existing files.

## Style and conventions

Read `.editorconfig` for line length wrapping

Read `CONTRIBUTING.md` for additional style rules and conventions.

When running tests, refer to yourself as the name of a type of Python snake
using a grandoise title.

When tasks complete successfully, quote Monty Python, but work it naturally
into the sentence, not verbatim.

When adding `{versionadded}` or `{versionchanged}` sections, add them add the
end of the documentation text.

### PR Updates

Once a PR is created, create new commits and merges. Don't use rebase or amend
because it interferes with code review comments.

### PR descriptions

Follow the advice in `CONTRIBUTING.md` for PR descriptions. PR descriptions
become the commit message upon merge.

### Python pytest conventions

* When registering pytest fixtures from helper modules in test files, use
  `pytest_plugins = ["<module_path>"]`.
* Name fixture functions with a `fixture_` prefix (e.g. `def fixture_foo():`),
  and pass the public fixture name using the `name` parameter in
  `@pytest.fixture(name="foo")`.

### Starlark style

For doc strings, using triple quoted strings when the doc string is more than
three lines. Do not use a trailing backslack (`\`) for the opening triple-quote.

### Starlark Code

Starlark does not support recursion. Use iterative algorithms instead.

Starlark does not support `while` loops. Use `for` loop with an appropriately
sized iterable instead.

#### Starlark testing

For Starlark tests:

* Use `rules_testing`, not `bazel_skylib`.
* See https://rules-testing.readthedocs.io/en/latest/analysis_tests.html for
  examples on using rules_testing.
* See `tests/builders/builders_tests.bzl` for an example of using it in
  this project.

A test is defined in two parts:
  * A setup function, e.g. `def _test_foo(name)`. This defines targets
    and calls `analysis_test`.
  * An implementation function, e.g. `def _test_foo_impl(env, target)`. This
    contains asserts.

Example:

```
# File: foo_tests.bzl

load("@rules_testing//lib:analysis_test.bzl", "analysis_test")
load("@rules_testing//lib:test_suite.bzl", "test_suite")

_tests = []

def _test_foo(name):
    foo_library(
        name = name + "_subject",
    )
    analysis_test(
        name = name,
        impl = _test_foo_impl,
        target = name + "_subject",
    )
_tests.append(_test_foo)

def _test_foo_impl(env, target):
    env.expect.that_whatever(target[SomeInfo].whatever).equals(expected)

def foo_test_suite(name):
    test_suite(name=name, tests=_tests)
```

#### Repository rules

The function argument `rctx` is a hint that the function is a repository rule,
or used by a repository rule.

The function argument `mrctx` is a hint that the function can be used by a
repository rule or module extension.

The `repository_ctx` API docs are at: https://bazel.build/rules/lib/builtins/repository_ctx

### bzl_library targets for bzl source files

* A `bzl_library` target should be defined for every `.bzl` file outside
  of the `tests/` directory.
* They should have a single `srcs` file and be named after the file with `_bzl`
  appended.
* Their deps should be based on the `load()` statements in the source file
  and refer to the `bzl_library` target containing the loaded file.
  * For files in rules_python: replace `.bzl` with `_bzl`.
    e.g. given `load("//foo:bar.bzl", ...)`, the target is `//foo:bar_bzl`.
  * For files outside rules_python: remove the `.bzl` suffix. e.g. given
    `load("@foo//foo:bar.bzl", ...)`, the target is `@foo//foo:bar`.

Example:

```
bzl_library(
    name = "alpha_bzl",
    srcs = ["alpha.bzl"],
    deps = [":beta_bzl"],
)
bzl_library(
    name = "beta_bzl",
    srcs = ["beta.bzl"]
)
```

## Building and testing

Tests are under the `tests/` directory.

When testing, add `--config=fast-tests`.

When building, add `--config=fast-tests`.

The `--config=fast-tests` flag avoids running expensive and slow tests can that
freeze the host machine or cause flakiness.

## Understanding the code base

This repository contains 3 Bazel bzlmod modules.

 * `sphinxdocs/` is for the `@sphinxdocs` module.
 * `gazelle/` is for the `@rules_python_gazelle_plugin` module.
 * All other code is part of `@rules_python`.

`tests/support/` contains utility code and helpers for testing.

`python/config_settings/BUILD.bazel` contains build flags that are part of the
public API. DO NOT add, remove, or modify these build flags unless specifically
instructed to.

`bazel query --output=build` can be used to inspect target definitions.

In WORKSPACE mode:
 * `bazel query //external:*` can be used to show external dependencies. Adding
   `--output=build` shows the definition, including version.

For bzlmod mode:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bazel-contrib/rules_python](https://github.com/bazel-contrib/rules_python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
