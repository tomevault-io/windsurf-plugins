---
trigger: always_on
description: handles the same step during a wheel build.
---

# Notes for LLM contributors

## Rule zero: prove it works before opening the PR

**Your job is to deliver code that is proven to work.** If you
have not proven the change works, it is not time to open the PR
yet. "It compiles", "type checks pass", and "the diff looks
right" are not proof. Proof is: the relevant tests run locally
and pass, the new behaviour is exercised by a test you added or
extended, and any user-visible path you touched has been
executed end-to-end. If you cannot run the suite in your
environment, say so explicitly in the PR body rather than
implying coverage you did not actually achieve. Opening a PR
that turns out not to work wastes the reviewer's time and is
the single fastest way to lose trust on this repo.

The rest of this document covers how to dress up that proven
change for review. None of it matters if rule zero is not met.

---

Read this before opening a pull request against
`aio-libs/frozenlist`. Agents keep getting the same things
wrong in this repo, so the rules below are not optional. If
you are about to skip a section because it sounds boilerplate,
that is exactly the section to re-read.

Human-facing contributor docs live under
[docs/contributing/](docs/contributing/) and
[CHANGES/README.rst](CHANGES/README.rst); this file is the
short orientation for agents.

## What this project is

`frozenlist` provides `FrozenList`, a list-like
`collections.abc.MutableSequence` that becomes immutable (and
hashable) once `freeze()` is called. It is a small, widely
deployed dependency of `aiohttp` and the rest of the
`aio-libs` stack, so it is performance sensitive. The
pure-Python implementation lives in `frozenlist/__init__.py`
and the Cythonized fast path lives in
`frozenlist/_frozenlist.pyx`; the package picks the compiled
class when the extension is importable and falls back to the
pure-Python `PyFrozenList` otherwise (or when the
`FROZENLIST_NO_EXTENSIONS` environment variable is set).

Useful entry points:

| Path                          | What                                                   |
| ----------------------------- | ------------------------------------------------------ |
| `frozenlist/__init__.py`      | pure-Python `FrozenList`, public API, extension loader |
| `frozenlist/__init__.pyi`     | type stubs for the public API                          |
| `frozenlist/_frozenlist.pyx`  | Cython `FrozenList` (hot path; ships in wheels)        |
| `tests/test_frozenlist.py`    | pytest suite (covers both implementations)             |
| `CHANGES/`                    | towncrier news fragments, one per PR                   |
| `docs/`                       | Sphinx docs source                                     |
| `packaging/pep517_backend/`   | in-tree PEP 517 backend that drives `cythonize`        |

## Pull request rules

These are the rules agents most often violate. Treat them as
mandatory.

### 1. Use the aio-libs pull request template

`frozenlist` ships its own
[`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md)
following the standard `aio-libs` shape. Do not invent your
own `## What / ## Why / ## How / ## Testing` layout; that is
the marker that the PR was written by an agent without reading
the conventions.

Fill out the template verbatim, like so:

```markdown
<!-- Thank you for your contribution! -->

## What do these changes do?

<short prose describing the change>

## Are there changes in behavior for the user?

<yes or no, plus a sentence if yes>

## Related issue number

Fixes #NNNN
<!-- or a bare reference if the change is related but does not close -->

## Checklist

- [x] I think the code is well written
- [x] Unit tests for the changes exist
- [x] Documentation reflects the changes
- [ ] If you provide code modifications, please add yourself to `CONTRIBUTORS.txt`
- [x] Add a new news fragment into the `CHANGES` folder
```

Tick the boxes that actually apply. If a row does not apply
(e.g. CI-only change with no tests), write `N/A` next to it
rather than silently leaving it blank.

### 2. Add a CHANGES fragment

Every user-visible PR needs a towncrier news fragment in
`CHANGES/`, named `<pr_number>.<category>.rst`. Categories
(defined in [CHANGES/README.rst](CHANGES/README.rst) and
[`towncrier.toml`](towncrier.toml)):

| Category       | When to use                                                     |
| -------------- | --------------------------------------------------------------- |
| `bugfix`       | corrects undesired behaviour                                    |
| `feature`      | new public API or behaviour                                     |
| `deprecation`  | announces a future removal                                      |
| `breaking`     | removes or changes something public in a breaking way           |
| `doc`          | documentation structure or build process                        |
| `packaging`    | downstream-visible packaging or build changes                   |
| `contrib`      | contributor experience (CI, dev env, test invocation)           |
| `misc`         | does not fit any of the above                                   |

Conventions for the fragment body:

- Use the past tense (`Fixed`, `Added`, `Bumped`), since it is
  read as a "what changed since the previous release" digest.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aio-libs/frozenlist](https://github.com/aio-libs/frozenlist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
