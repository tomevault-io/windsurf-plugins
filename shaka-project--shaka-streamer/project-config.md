---
trigger: always_on
description: Shaka Streamer is a Python 3 CLI tool and library that wraps FFmpeg and Shaka
---

# Shaka Streamer - Agent Guide

Shaka Streamer is a Python 3 CLI tool and library that wraps FFmpeg and Shaka
Packager into a config-file-based workflow for preparing streaming media (HLS
and DASH, both VOD and live). Users write YAML config files; the tool handles
transcoding, packaging, and optional cloud upload.

## Attribution

Read [AGENT-ATTRIBUTION.md](AGENT-ATTRIBUTION.md) for attribution details.

## Repo Layout

```
streamer/              Main Python package - pipeline nodes, config system, CLI logic
  *_configuration.py   Config schema classes (metaprogramming - see Config System below)
  node_base.py         Base class for all pipeline nodes
  controller_node.py   Top-level orchestrator; entry point for library use
  transcoder_node.py   FFmpeg wrapper
  packager_node.py     Shaka Packager wrapper
  cloud/               Cloud upload nodes (one per vendor)
tests/
  tests.js             Karma/Jasmine browser-based test specs
  karma.conf.js        Karma configuration
run_end_to_end_tests.py  Test runner: Flask server + karma + browser playback
config_files/          Sample YAML configs for users
docs/                  Sphinx documentation source; built from source + docstrings
binaries/              Builds the shaka-streamer-binaries pip package locally
shaka-streamer         CLI entry point script
```

## Environment Setup

Requires Python 3.9+. On Linux, also install system deps first:

```sh
sudo apt install libva2 libva-drm2 nodejs npm xvfb
```

Install Python dependencies:

```sh
pip3 install -r requirements.txt
pip3 install -r optional_requirements.txt   # mypy, flask, sphinx, etc.
```

Build and install the local binary package (FFmpeg + Shaka Packager):

```sh
python3 binaries/build_wheels.py

# Then install the wheel for your platform:
# Linux x64:    pip3 install binaries/dist/shaka_streamer_binaries*linux*x86_64.whl
# Linux arm64:  pip3 install binaries/dist/shaka_streamer_binaries*linux*aarch64.whl
# macOS Intel:  pip3 install binaries/dist/shaka_streamer_binaries*mac*x86_64.whl
# macOS arm64:  pip3 install binaries/dist/shaka_streamer_binaries*mac*arm64.whl
# Windows:      pip3 install binaries/dist/shaka_streamer_binaries*win*amd64.whl
```

## Commands

| Task | Command |
|------|---------|
| Type check only | `npm run mypy` |
| Lint only | `npm run pylint` |
| Lint + type check | `npm run lint` |
| Run tests (Linux) | `xvfb-run -a python3 run_end_to_end_tests.py` |
| Run tests (other) | `python3 run_end_to_end_tests.py` |
| Build docs | `bash docs/build.sh` |

Mypy and pylint are the only fast verifications.  A full test run requires the
complete environment described above.

## Testing

The test suite is end-to-end only - there are no unit tests. Each test:

1. Starts a local Flask server (`run_end_to_end_tests.py`)
2. Launches Karma, which runs Jasmine specs in a browser
3. The browser starts Shaka Streamer via HTTP, plays back the resulting stream,
   and asserts on playback success

**Infrastructure required:** installed binaries (FFmpeg + Shaka Packager),
Node.js/npm, a browser, and xvfb on Linux for headless operation.

**Test assets** are downloaded automatically from Google Cloud Storage the
first time tests run. Do not check in media files.

**When to update tests:**
- Changed behavior in an existing feature: update the relevant spec in `tests/tests.js`
- New feature: add new Jasmine `it()` blocks covering the happy path and key edge cases
- Bug fix for something not caught by existing tests: add a regression test

## Config System

> **This is the highest-risk area of the codebase. Read carefully before
> modifying any `*_configuration.py` file.**

The classes in `streamer/*_configuration.py` use metaprogramming to serve
three purposes simultaneously:

1. **Runtime validation** - field types and constraints are checked when a user
   loads a YAML config
2. **Static type checking** - mypy uses the same definitions to type-check
   internal code
3. **Public documentation** - Sphinx extracts field descriptions from these
   classes for the hosted docs

Adding or modifying a config field incorrectly can silently break one or more
of these without an obvious error. Before touching any `*_configuration.py`
file:

- Read at least two or three of the existing `*_configuration.py` files to
  understand the patterns in use
- Follow those patterns exactly for new fields - do not invent new approaches
- Update the docs after any config schema change (`bash docs/build.sh` and
  review the output)
- Backwards compatibility matters: removing or renaming fields is a breaking
  change for existing user configs

## Architecture

Shaka Streamer processes media through a directed pipeline of **nodes**. Each
node runs as a subprocess or thread and communicates with adjacent nodes via
OS pipes. The key nodes are:

- `ControllerNode` - orchestrates the pipeline; the main entry point for
  library users
- `TranscoderNode` - wraps FFmpeg
- `PackagerNode` - wraps Shaka Packager
- `cloud/` nodes - upload output to a cloud storage provider

**New node types are rare and require strong justification.** The node roles
are intentionally fixed. The only common case for a new node is a new cloud
upload destination. If you think you need a new node, consider whether the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shaka-project/shaka-streamer](https://github.com/shaka-project/shaka-streamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
