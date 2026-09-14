---
trigger: always_on
description: Bluetoe is a C++11 Bluetooth Low Energy peripheral stack for very small microcontrollers:
---

# Bluetoe

Bluetoe is a C++11 Bluetooth Low Energy peripheral stack for very small microcontrollers:
a template-based GATT server, ATT/L2CAP, a link layer that runs directly on the radio,
a security manager (legacy and LESC pairing), and hardware bindings for Nordic nRF51/nRF52.
Everything that can be decided at compile time is; RAM and Flash Memory usage footprint is the primary design driver.

Sole author and maintainer: Torsten Robitzki. Treat every change as something the maintainer
must be able to review and understand.

## How to work in this repository

- Discuss before changing. Propose an approach and wait for agreement before editing tracked
  files, unless the task was explicitly scoped as "just do it".
- Work on `master`. Feature branches are only used when the maintainer asks for one.
- Show every diff. The maintainer wants to read and learn from each change before it is committed.
  Present the full diff in the conversation, wait for approval, then commit. Never commit or push
  unreviewed changes. (This rule may be relaxed later; do not assume it has been.)
- Small steps. One logical change per commit.
- Never push, open pull requests, or create GitHub issues without being asked.
- Do not run `git` commands that rewrite history or discard work (`reset --hard`, `checkout --`,
  `clean`, `push --force`, branch deletion).
- One git step per command. Never chain checkout, stash, merge, commit and push into a single
  shell command: a failure in the middle (e.g. a stash conflict) does not stop the rest, and
  `git add` on a conflicted file silently commits the conflict markers. Check `git status` between steps.
- Explain the "why" of a change in plain language, not only the diff.
- Stay on C++11. Modernisation (C++14/17 features, simplifying the meta-programming) is a
  deliberate, separate project for a later session. Do not propose or sneak in newer language
  features as part of unrelated changes.
- The exception is the scheduled radio 2 work: the new radio interface, `abs_time`, the test rig,
  the tester and their host side tooling may use any standard that helps. It does not extend to the
  existing link layer, the GATT layer or the current bindings. See
  `documentation/scheduled_radio_test_rig.md`.

## Layout

- `bluetoe/*.hpp` – GATT server, services, characteristics, ATT, L2CAP (public API root: `server.hpp`)
- `bluetoe/link_layer/` – link layer, advertising, connection procedures, PDU buffers
- `bluetoe/sm/` – security manager
- `bluetoe/utility/` – meta-programming tools, attribute table, bit helpers
- `bluetoe/services/` – predefined services (DIS, BAS, CSC, HID, bootloader)
- `bluetoe/bindings/nordic/` – nRF51/nRF52 radio bindings, vendored micro-ecc
- `bluetoe/hci/` – HCI-based link layer (currently a stub)
- `tests/` – Boost.Test unit and protocol tests, host-only, with a simulated radio in `tests/test_tools/`
- `platforms/` – what every firmware is built on: the ARM toolchain file, per-binding startup code,
  linker scripts and flash commands, the C++ runtime, `assert()`, the build container
- `examples/` – nRF52 firmware examples, cross-compiled with arm-none-eabi-gcc
- `documentation/` – Doxygen input; API docs are published at https://torstenrobitzki.github.io/bluetoe/

## Building and testing

Host build with unit tests (requires CMake, Boost headers, a C++11 compiler). The primary test
configuration is **Debug**: the library contains several hundred `assert()` calls that are the
first line of defence against protocol and buffer errors, and a Release build defines `NDEBUG`,
which compiles all of them out.

```bash
cmake -S . -B build -DBLUETOE_BUILD_UNIT_TESTS=ON -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j8
ctest --test-dir build -j8
```

Before declaring a change done, also build and run the tests once in **Release**
(`-DCMAKE_BUILD_TYPE=Release`, e.g. in a `build_release` directory). This proves that no behaviour
depends on the debug instrumentation, e.g. a side effect inside an `assert()` expression, or code
that only works because a check fired first. Both configurations must pass.

- Each `tests/*_tests.cpp` file is a separate executable, registered in `tests/CMakeLists.txt`
  with `add_and_register_test(<name>)`. A new test file must be added there.
- Tests are compiled with `-Wall -pedantic -Wextra -Wfatal-errors`. On Clang, AddressSanitizer is
  enabled for all build types (`tests/CMakeLists.txt`); on GCC there is currently no sanitizer.
  CI compiles C++ with `-Werror`, so every warning is a build failure there.
- Run only the affected test executable while iterating, e.g. `./build/tests/link_layer/ll_connection_tests`;
  run the full `ctest` before declaring a change done.
- `-DBLUETOE_EXCLUDE_SLOW_TESTS=ON` skips the long-running link layer tests.
- Firmware examples need an ARM toolchain and `NRF5_SDK_ROOT`; see `platforms/README.md` and
  `platforms/docker/` for a reproducible container. Do not attempt to build them on the host.

## Continuous integration

`.github/workflows/tests.yml` runs on every push and pull request:

- Unit tests on Linux with GCC and Clang, each in Debug and Release, and on macOS in Debug.
  Linux is the strict platform: Apple's standard library includes more headers transitively,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TorstenRobitzki/bluetoe](https://github.com/TorstenRobitzki/bluetoe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
