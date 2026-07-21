---
trigger: always_on
description: **Your job is to deliver code that is proven to work.** If you
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

Read this before opening a pull request against `aio-libs/yarl`.
Agents keep getting the same things wrong in this repo, so the
rules below are not optional. If you are about to skip a section
because it sounds boilerplate, that is exactly the section to
re-read.

Human-facing contributor docs live under
[docs/contributing/](docs/contributing/) and
[CHANGES/README.rst](CHANGES/README.rst); this file is the short
orientation for agents.

## What this project is

`yarl` is the URL parsing and building library used by `aiohttp`
and the rest of the `aio-libs` stack. It is small, widely
deployed, and performance sensitive. The pure-Python code lives
under `yarl/` and the quoting hot path is Cythonized
(`yarl/_quoting_c.pyx`, with a pure-Python equivalent at
`yarl/_quoting_py.py`).

Useful entry points:

| Path                    | What                                                   |
| ----------------------- | ------------------------------------------------------ |
| `yarl/_url.py`          | `URL` class, all public API                            |
| `yarl/_parse.py`        | `split_url` / `split_netloc`, RFC 3986 parsing         |
| `yarl/_path.py`         | path normalisation helpers                             |
| `yarl/_query.py`        | query string assembly / encoding                       |
| `yarl/_quoting.py`      | dispatcher between the C and Python quoters            |
| `yarl/_quoting_c.pyx`   | Cython quoter (hot path; ships in wheels)              |
| `yarl/_quoting_py.py`   | pure-Python quoter (used when the extension is absent) |
| `yarl/_quoters.py`      | pre-built quoter instances reused across the codebase  |
| `tests/`                | pytest suite, including the benchmarks                 |
| `CHANGES/`              | towncrier news fragments, one per PR                   |
| `docs/`                 | Sphinx docs source                                     |
| `packaging/`            | in-tree PEP 517 backend that drives `cythonize`        |

## Pull request rules

These are the rules agents most often violate. Treat them as
mandatory.

### 1. Use the aio-libs pull request template

`yarl` follows the standard `aio-libs` PR template. Even though
the repo does not ship its own `.github/PULL_REQUEST_TEMPLATE.md`,
maintainers expect every PR body to follow this exact structure.
Do not invent your own `## What / ## Why / ## How / ## Testing`
layout; that is the marker that the PR was written by an agent
without reading the conventions.

Fill out the template verbatim, like so:

```markdown
<!-- Thank you for your contribution! -->

## What do these changes do?

<short prose describing the change>

## Are there changes in behavior for the user?

<yes or no, plus a sentence if yes>

## Is it a substantial burden for the maintainers to support this?

<no, plus a sentence on why if relevant>

## Related issue number

Fixes #NNNN
<!-- or a bare reference if the change is related but does not close -->

## Checklist

- [x] I think the code is well written
- [x] Unit tests for the changes exist
- [x] Documentation reflects the changes
- [ ] If you provide code modification, please add yourself to `CONTRIBUTORS.txt`
- [x] Add a new news fragment into the `CHANGES/` folder
```

Tick the boxes that actually apply. If a row does not apply
(e.g. CI-only change with no tests), write `N/A` next to it
rather than silently leaving it blank.

For a real filled-out example in this repo, see
[aio-libs/yarl#1681](https://github.com/aio-libs/yarl/pull/1681).

### 2. Add a CHANGES fragment

Every user-visible PR needs a towncrier news fragment in
`CHANGES/`, named `<pr_number>.<category>.rst`. Categories
(defined in [CHANGES/README.rst](CHANGES/README.rst)):

| Category       | When to use                                                     |
| -------------- | --------------------------------------------------------------- |
| `bugfix`       | corrects undesired behaviour                                    |
| `feature`      | new public API or behaviour                                     |
| `deprecation`  | announces a future removal                                      |
| `breaking`     | removes or changes something public in a breaking way           |
| `doc`          | documentation structure or build process                        |
| `packaging`    | downstream-visible packaging or build changes                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aio-libs/yarl](https://github.com/aio-libs/yarl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
