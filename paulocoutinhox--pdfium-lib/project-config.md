---
trigger: always_on
description: This file records how the project is organised and how work on it is expected to be done.
---

# Project conventions

This file records how the project is organised and how work on it is expected to be done.
Read it before changing anything.

## What this project is

It compiles Google's PDFium into a distributable library for several platforms. It does not
fork PDFium and it does not vendor its source. Every build clones PDFium from upstream,
applies the minimum set of patches needed for the target platform, compiles it and stages
the result. All the value of the project sits in the tooling that makes those steps
reproducible, so keep the tooling clean and never let a fix drift into a copy of upstream code.

## Layout

```
make.py                    Task entry point, one task per platform step.
modules/
  config.py                Every configurable value: pdfium branch, targets, configurations.
  common.py                Shared tasks and the gn argument builder.
  pdfium.py                Cloning and syncing pdfium through gclient.
  patch.py                 Reusable patches applied to the checkout.
  <platform>.py            The six tasks of one platform.
docs/BUILD_<PLATFORM>.md   Build tutorial for one platform.
.github/workflows/         One workflow per platform, all with the same shape.
sample/                    Console sample used by the test task on desktop platforms.
sample-apple/              Xcode sample for macOS and iOS.
sample-wasm/               Sample for the WASM build.
extras/wasm/               Files shipped with the WASM package, including the viewer template.
build/                     Output only, ignored by git, safe to delete at any time.
```

`build/` holds three different things and the names repeat, so read paths carefully:

```
build/<platform>/pdfium/          The pdfium checkout.
build/<platform>/pdfium/build/    chromium/src/build.git, pulled by pdfium's DEPS.
build/<platform>/<config>/        The staged library and headers, what gets archived.
```

## Platform modules

Every platform module exposes the same six tasks, in this order, and nothing else:

| Task | What it does |
| --- | --- |
| `run_task_build_pdfium` | Clones and syncs pdfium through `modules/pdfium.py`. |
| `run_task_patch` | Applies the patches the platform needs. Must be idempotent. |
| `run_task_build` | Runs `gn gen` and `ninja` for every configuration and target. |
| `run_task_install` | Stages the library and the headers under `build/<platform>/<config>`. |
| `run_task_test` | Builds the sample against the staged library and runs it. |
| `run_task_archive` | Packs the staged directory into `<platform>.tgz`. |

Use `modules/macos.py` as the reference when adding a platform. Register the tasks in
`make.py`, in the docstring and in the dispatch chain, keeping the platform sections in the
same order in both.

## Configuration

`modules/config.py` is the single source of truth. A value that lives there is never
repeated anywhere else, and code reads it instead of assuming what it holds. Each platform
declares three things:

```python
configurations_<platform> = ["release"]
shared_lib_<platform> = False
targets_<platform> = [
    {"target_os": "linux", "target_cpu": "x64", "pdfium_os": "linux"},
]
```

`target_os` names the output directory, `pdfium_os` is the value gn expects, and they differ
on purpose: `macos` is `mac` to gn, `windows` is `win`. Adding an architecture is one entry
in the list and nothing else.

Build arguments belong in `get_build_args` in `modules/common.py`, never inline in a platform
module. Static builds get `pdf_is_complete_lib=true`, and every platform needs
`use_custom_libcxx=false`, because the bundled libc++ never reaches a static archive and its
symbols carry an ABI namespace no system runtime provides.

Apple platforms ship a static library, while Android, Linux and Windows ship a shared one.
The two are not interchangeable: `apply_public_headers` makes `FPDF_EXPORT` unconditional, so
the staged headers declare `__declspec(dllimport)` and only work against the shared library.

## Patches

PDFium and the chromium build system it pulls in assume the toolchain Google packages
internally. Where that assumption does not hold, the patch task rewrites the checkout before
building. Rules for patches:

- Live in `modules/patch.py`, and are called from a platform's `run_task_patch`.
- Detect what the machine actually has and adapt to it. Never hardcode a version, a path or a tool location.
- Are idempotent, because CI runs the patch task twice to prove it. Report `Applied` the first time and `Skipped` afterwards.
- Are a no-op where they do not apply, so a platform module can call them unconditionally.
- Touch only files under `build/`, which is disposable. Never patch anything tracked by this repository.

The windows patches are the reference: chromium pins the SDK version in `vs_toolchain.py` and
`setup_toolchain.py`, and pins the API level in `config/win/BUILD.gn`. Both are rewritten from
what is installed on the machine.

## Continuous integration

One workflow per platform, all built from the same shape: set up Python, CMake and Ninja,
install the requirements, clone depot tools, put it on `PATH`, then run the six tasks in
order, upload the archive, and deploy it on a tag. Keep new workflows aligned with the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulocoutinhox/pdfium-lib](https://github.com/paulocoutinhox/pdfium-lib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
