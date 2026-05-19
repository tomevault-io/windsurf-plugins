---
trigger: always_on
description: This document is a guide for AI coding agents working in this repository. It
---

# AGENTS.md — projspec

This document is a guide for AI coding agents working in this repository. It
covers the architecture of the `projspec` package (located in `src/projspec/`),
the three central class families, the contract every `parse()` method must
honour, and the conventions used throughout.

Extensions and the Qt application (`vsextension/`, `pycharm_plugin/`,
`src/projspec/qtapp/`) are out of scope. See `vsextension/ACTIONS.md` for
the VSCode extension specification.

---

## Repository layout

```
src/projspec/
  __init__.py          # public re-exports: Project, ProjectSpec, get_cls
  proj/
    base.py            # Project, ProjectSpec, ProjectExtra, ParseFailed
    *.py               # one file per concrete spec type
  content/
    base.py            # BaseContent + content registry
    *.py               # one file per concrete content type
  artifact/
    base.py            # BaseArtifact, FileArtifact + artifact registry
    *.py               # one file per concrete artifact type
  utils.py             # AttrDict, camel_to_snake, run_subprocess, …
  config.py            # get_conf / set_conf
vsextension/           # a UI for vscode, calling projspec as a subprocess
qtapp/                 # standalone UI on pyqt5, calling projspec in-process
tests/
  conftest.py          # shared fixtures (proj = Project("/data"))
  test_basic.py        # smoke tests
  test_roundtrips.py   # serialise / deserialise round-trips
  …
```

---

## The three class families

### 1. `Project`  (`proj/base.py:43`)

The top-level container for a parsed directory.  It is not subclassed.

Key attributes set during `__init__` → `resolve()`:

| attribute | type | description |
|-----------|------|-------------|
| `specs` | `AttrDict` | matched `ProjectSpec` instances, keyed by snake-case class name |
| `contents` | `AttrDict` | `BaseContent` instances contributed by `ProjectExtra` specs |
| `artifacts` | `AttrDict` | `BaseArtifact` instances contributed by `ProjectExtra` specs |
| `children` | `AttrDict` | child `Project` instances found by directory walking |
| `fs` | `fsspec.AbstractFileSystem` | filesystem used for all file I/O |
| `url` | `str` | FS-normalised path to the project root |
| `basenames` | `dict[str, str]` | `{basename: full_path}` for every entry at the root |
| `pyproject` | `dict` | parsed `pyproject.toml`, or `{}` |

`Project.resolve()` iterates every registered `ProjectSpec` subclass and calls
`cls(proj)` (which runs `match()`) then `inst.parse()`.  A `ValueError` /
`ParseFailed` means the directory did not match that type and is silently
skipped.  Any other exception is logged but does not abort parsing.

`ProjectExtra` subclasses are handled differently: their `contents` and
`artifacts` are merged directly into `proj.contents` / `proj.artifacts` rather
than being stored in `proj.specs`.

---

### 2. `ProjectSpec`  (`proj/base.py:435`)

Base class for every concrete project type.  Subclasses are **auto-registered**
on import via `__init_subclass__` using their snake-case name as the key
(`proj/base.py:511`).

Lifecycle inside `Project.resolve()`:

```
cls(proj)          ← __init__ calls self.match(); raises ParseFailed if False
inst.parse()       ← populate self._contents and self._artifacts
```

Important class-level attribute:

| attribute | description |
|-----------|-------------|
| `spec_doc` | URL to upstream specification docs (optional but encouraged) |

Instance attributes after `parse()`:

| attribute | type | description |
|-----------|------|-------------|
| `_contents` | `AttrDict` | content objects for this spec |
| `_artifacts` | `AttrDict` | artifact objects for this spec |
| `proj` | `Project` | back-reference to the owning project |

Public properties `.contents` and `.artifacts` delegate to `_contents` /
`_artifacts` and call `parse()` lazily if they are `None` (`proj/base.py:466`).

#### `ProjectExtra`  (`proj/base.py:542`)

A special subclass of `ProjectSpec` for cross-cutting concerns (CI/CD, Docker,
pre-commit, requirements files, …).  These specs are *not* standalone projects.
After parsing, `Project.resolve()` merges their `contents` / `artifacts` into
the root project rather than storing them in `proj.specs`.

---

### 3. `BaseContent`  (`content/base.py:11`)

A **dataclass** holding descriptive information extracted from a project.
Content objects are read-only descriptions; they have no executable behaviour.

Every subclass is a `@dataclass` that **must** include `proj: Project` as its
first field (inherited from `BaseContent`).

Subclasses are auto-registered on import via `__init_subclass__` (keyed by
snake-case name).

Concrete content classes:

| class | module | fields |
|-------|--------|--------|
| `Environment` | `content/environment.py` | `stack: Stack`, `precision: Precision`, `packages: list[str]`, `channels: list[str]` |
| `Command` | `content/executable.py` | `cmd: list[str] \| str` |
| `DescriptiveMetadata` | `content/metadata.py` | `meta: dict[str, str]` |
| `License` | `content/metadata.py` | `shortname`, `fullname`, `url` |
| `PythonPackage` | `content/package.py` | `package_name: str` |
| `RustModule` | `content/package.py` | `name: str` |
| `NodePackage` | `content/package.py` | `name: str` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fsspec/projspec](https://github.com/fsspec/projspec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
