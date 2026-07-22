---
trigger: always_on
description: This file defines the protocol for porting GCC libstdc++ source and configure
---

# libstdc++ Bazel Porting Guide

This file defines the protocol for porting GCC libstdc++ source and configure
logic into Bazel. It applies to files under `3rd_party/gcc/libstdcxx` and the
matching GCC source declarations in `3rd_party/gcc/gcc.BUILD.bazel`.

The scope here is the libstdc++ Bazel port itself: source lists, generated
headers, header overlays, configure checks, and configure-derived policy. This
file does not describe C++ standard library selection, cc toolchain declaration,
or downstream toolchain integration.

The GCC source graph is versioned. `3rd_party/gcc/version.bzl` is the source of
truth for `GCC_VERSIONS`, release metadata, constraint names, and version helper
functions. The module extension materializes one concrete GCC repository per
version and a reproducible `@gcc` trampoline repository that selects the right
concrete target from the active `//constraints/cxxstdlib:libstdcxx.<version>`
constraint. Keep helper `.bzl` files in the main repository and load them from
the concrete GCC build with `@llvm//...`; `3rd_party/gcc/gcc.BUILD.bazel`
should remain the BUILD file loaded inside each concrete GCC source repository.

## Core Rule

Do not port checks by memory or by manually scanning one file. First build a
mechanical inventory from the GCC sources, then classify every item, then port
the active supported behavior, then validate that the inventory and Bazel model
still agree.

Every GCC check, macro, define, substitution, conditional, and target branch
must end up in exactly one of these states:

- `probe-modeled`: represented by a Bazel compile, link, declaration, header,
  or other toolchain-backed probe.
- `policy-modeled`: represented by a fixed or target-derived Bazel policy.
- `target-derived`: selected from Bazel target/platform information.
- `build-setting-later`: currently fixed, but should become an explicit private
  build setting before claiming full knob parity.
- `not-needed`: configure/build/install/testsuite plumbing that Bazel replaces.
- `unsupported-target`: target family not supported by this libstdc++ port.
- `unsupported-feature`: optional libstdc++ feature not built by this port.

Avoid a vague `unsupported` state. Say whether the item is unsupported because
the target family is out of scope or because the feature is out of scope.

## Source Counterparts

Keep Bazel files shaped like the GCC files they port:

- `3rd_party/gcc/libstdcxx/configure.ac.bzl` is the active
  counterpart of `libstdc++-v3/configure.ac`. It composes the supported
  configure flow.
- `3rd_party/gcc/libstdcxx/acinclude.m4.bzl` is the counterpart of
  `libstdc++-v3/acinclude.m4`.
- `3rd_party/gcc/libstdcxx/crossconfig.m4.bzl` is the counterpart of
  `libstdc++-v3/crossconfig.m4`.
- `3rd_party/gcc/libstdcxx/gcc_config_checks.bzl` is the counterpart
  for GCC top-level `config/*.m4` checks used by libstdc++.
- `3rd_party/gcc/libstdcxx/linkage.m4.bzl` is the counterpart of
  `libstdc++-v3/linkage.m4`.
- `3rd_party/gcc/libstdcxx/autoconf/checks.bzl`,
  `3rd_party/gcc/libstdcxx/autoconf/autoconf_config.bzl`,
  `3rd_party/gcc/libstdcxx/autoconf/autoconf_hdr.bzl`,
  `3rd_party/gcc/libstdcxx/autoconf/cc_configure_probe.bzl`, and
  `3rd_party/gcc/libstdcxx/autoconf/providers.bzl` are local generic
  autoconf mechanics. They should stay free of libstdc++ source-policy
  decisions so a future external autoconf ruleset migration is a thin adapter
  change.
- `libstdc++-v3/linkage.m4` helpers are represented in
  `3rd_party/gcc/libstdcxx/linkage.m4.bzl`, even when they are generic math or
  stdlib declaration/linkage checks.
- `3rd_party/gcc/libstdcxx/target_config.bzl`,
  `3rd_party/gcc/libstdcxx/libstdcxx_cxxconfig_header.bzl`,
  `3rd_party/gcc/libstdcxx/libstdcxx_gthr_headers.bzl`,
  `3rd_party/gcc/libstdcxx/libstdcxx_largefile_config_header.bzl`,
  `3rd_party/gcc/libstdcxx/libstdcxx_symbols_version_script.bzl`, and
  `3rd_party/gcc/libstdcxx/BUILD.bazel` may consume configure-derived policy,
  but should not hide new configure semantics without updating the inventories.
  `runtimes/libstdcxx/BUILD.bazel` is only the public runtime facade.

Each source-counterpart `.bzl` file should start with a short comment naming
the GCC file(s) it was ported from. When a group of upstream macros is collapsed
into one Bazel helper, leave an anchor comment listing the upstream macro names.

## Required Tracking Files

Maintain these human-readable tracking files in `3rd_party/gcc/libstdcxx/docs`:

- `autoconf.checks.md`: checklist of check definitions available from GCC
  sources. This covers GCC top-level `config/*.m4` macros, libstdc++ custom
  `GLIBCXX_*` macros, and helper macros such as those in `linkage.m4`.
- `autoconf.usage.md`: checklist of configure usage, in the order
  `configure.ac` uses checks and macros. This file explains which definitions
  are actually reached for the current supported Linux GNU configuration and
  which branches are inactive.
- `autoconf.README.md`: glossary and report for the configure model. For every
  check, it records what the upstream check does, when it runs, what outputs it
  produces, and the Bazel status.

The lower-level machine status files, such as `config_define_status.txt` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hermeticbuild/hermetic-llvm](https://github.com/hermeticbuild/hermetic-llvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
