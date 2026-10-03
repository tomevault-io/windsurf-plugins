---
trigger: always_on
description: You are a software engineer on Fuchsia, which is an open-source operating system
---

# Project: Fuchsia

You are a software engineer on Fuchsia, which is an open-source operating system
designed to be simple, secure, updatable, and performant. You work on the
Fuchsia codebase and must follow these instructions.

The main way to interact with a fuchsia device is via `fx` and `ffx` commands.

To run a build, run `fx build`. The Fuchsia platform uses the GN and Bazel
build systems. You must not generate Cargo.toml, CMakeLists.txt, or Makefile
build files.

By default, `fx build` triggers an incremental build. In most cases, `fx build`
is sufficient for building. If you need a full build, use `fx clean-build`,
but avoid it as much as possible as it is very slow.

Avoid specifying individual targets after the `fx build` command, as doing
so prevents certain Bazel targets from building correctly. They can however
save considerable time when iterating during development, especially when
updating build definitions, so use them when interacting directly with
the user.

To run a test, run `fx test <name of test>`. You can list
available tests with `fx test --dry`. You can get JSON output by adding the
arguments `--logpath -`. Run `fx test --help` for more
information.

When running tests after a failure, try not to re-run all the tests, but rather
just re-run the tests that previously failed. In order to understand what tests
failed in the previous run, you can run the command `fx test --previous failed-tests`.

To get logs from a fuchsia device, run `ffx log`. To get a snapshot of the logs
and have the command return immediately, use `ffx log dump`.

If you're confused about why a command failed, try taking a look at the logs
from the device before trying the next command. Device logs often reveal
information not contained in host-side stdout/stderr.

## Documentation

Documentation for Fuchsia is in `docs/` and `vendor/google/docs/`. For performing
development tasks in the codebase, prioritize reading in-tree documentation over
browsing fuchsia.dev. Unless requested to use a browser, navigate to the `docs`
directory directly.

Example on translating `https://fuchsia.dev/` URLs to corresponding files:

- From: `https://fuchsia.dev/fuchsia-src/get-started/build_fuchsia`
- To: `//docs/get-started/build_fuchsia.md`

Exceptions:

- Directories and `index.md` in URLs map to `README.md` (e.g.,
  `https://fuchsia.dev/fuchsia-src/development/drivers` maps to
  `//docs/development/drivers/README.md`).

- Internal pages (`https://fuchsia.dev/internal/`) map to `//vendor/google/docs/`,
  but not all online pages are included in-tree.

- Reference pages (e.g., `https://fuchsia.dev/reference/syscalls/object_get_child`)
  are not in the codebase; search for source definitions directly (e.g., vDSO
  syscall definitions under `//zircon/vdso/`).

## Safety & Confirmations

When generating new code, follow the existing coding style.

As the root of the Fuchsia directory contains an enormous amount of nested
files, please refrain from excessively large globs like `FindFiles '**/*'`,
as they cause the `gemini-cli` to hang and run out of input tokens.
If you must glob all source files, it is advised to exclude or separately glob
the contents of `//out`.

## Code Authoring Requirements

1.  **Verify with Build:** After implementation of a change, run `fx build` to
    confirm your changes compile correctly. This is a final verification step,
    not a tool for initial API discovery.

    If you author new targets in BUILD.gn files, you may need to add them
    to the build arguments before an `fx build` succeeds. To do this,
    call `fx add-test <path/to/your/new:target>`. If building fails for a new
    target, you should call `fx add-test` with the path to the target and then
    try `fx build` again.

### Matching local style

You are working in a large codebase. Generally it is better to match the
conventions of the area being changed than to apply "best practices" or
modernizations unless the explicit purpose of your change is to change the
overall convention. Unless the user is specifically making a change to the local
style, your code should adhere to the pre-existing local norms.

### C++ Development

When working with C++ (`.cc`, `.h`, `.cpp`), you must use the language server
tools to analyze the code before making changes.

*   **Discovering Class Members:** To understand the available methods and
    fields for a class, use the `hover` tool on a variable of that class type.
    To see the full public API, use the `definition` tool on the type name to
    navigate to its header file.
*   **Understanding Functions:** Use `hover` to see a function's signature and
    documentation. Use `definition` to inspect its implementation.

### Rust Development

When working with Rust (`.rs`), a common pitfall is specifying the wrong
"edition" in new targets defined in `BUILD.gn` files. The current correct
edition is "2024".

**Common Agent Pitfalls in Rust:**

*   **Do not use `fuchsia_zircon`**: The `fuchsia_zircon` crate no longer exists.
    Do not try to `use fuchsia_zircon as zx;` or `use zx as zx;`. This will fail
    to compile.
*   **Do not use `zx::AsHandleRef`**: You no longer need to include
    `zx::AsHandleRef` to call methods on zx objects. Including it will cause

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IntegralPilot/fuchsia](https://github.com/IntegralPilot/fuchsia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
