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
executed end-to-end. For multidict specifically, that means the
suite passes under **both** the default C-extension build **and**
the pure-Python build (`MULTIDICT_NO_EXTENSIONS=1`); see the
[Tests](#tests) section for the exact commands. If you cannot
run the suite in your environment, say so explicitly in the PR
body rather than implying coverage you did not actually achieve.
Opening a PR that turns out not to work wastes the reviewer's
time and is the single fastest way to lose trust on this repo.

The rest of this document covers how to dress up that proven
change for review. None of it matters if rule zero is not met.

---

Read this before opening a pull request against `aio-libs/multidict`.
Agents keep getting the same things wrong in this repo, so the rules
below are not optional. If you are about to skip a section because it
sounds like boilerplate, that is the section to re-read.

Human-facing contributor docs live under
[CHANGES/README.rst](CHANGES/README.rst); this file is the short
orientation for agents.

## What this project is

`multidict` is the case-insensitive, multi-value mapping used by
`aiohttp`, `yarl`, and the rest of the `aio-libs` stack. It is small,
widely deployed, and performance sensitive. It ships **two parallel
implementations that must stay behaviourally identical**:

- Pure Python: `multidict/_multidict_py.py`
- C extension: `multidict/_multidict.c`, plus headers in
  `multidict/_multilib/`

Useful entry points:

| Path                                  | What                                                            |
| ------------------------------------- | --------------------------------------------------------------- |
| `multidict/__init__.py`               | public surface; chooses C vs. pure-Python impl at import time   |
| `multidict/_abc.py`                   | `MultiMapping`, `MutableMultiMapping`, type protocols           |
| `multidict/_multidict_py.py`          | pure-Python `MultiDict`, `CIMultiDict`, `istr`, proxies         |
| `multidict/_multidict.c`              | C implementation entry points and type definitions              |
| `multidict/_multilib/hashtable.h`     | core hash table (``md_*`` functions, resize, lookup)            |
| `multidict/_multilib/htkeys.h`        | key-storage layout, ``estimate_log2_keysize``                   |
| `multidict/_multilib/istr.h`          | C ``istr`` (case-insensitive str)                               |
| `multidict/_multilib/iter.h`          | views and iterators for the C impl                              |
| `multidict/_multilib/parser.h`        | argument parsing for ``extend`` / ``update`` / constructors     |
| `multidict/_multilib/pythoncapi_compat.h` | vendored upstream; do not edit                              |
| `tests/`                              | pytest suite, parametrised across both backends                 |
| `CHANGES/`                            | towncrier news fragments, one per PR                            |

`MULTIDICT_NO_EXTENSIONS=1` forces the pure-Python build at install
time; the default is the C extension. `MULTIDICT_DEBUG_BUILD=1` builds
the C extension with `-O0 -g3 -UNDEBUG`.

## Pull request rules

These are the rules agents most often violate. Treat them as mandatory.

### 1. Use the aio-libs pull request template

`multidict` follows the standard `aio-libs` PR template. Even though
this repo does not ship its own `.github/PULL_REQUEST_TEMPLATE.md`,
maintainers expect every PR body to follow the structure below.
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
<!-- or a bare reference if related but not closing -->

## Checklist

- [x] I think the code is well written
- [x] Unit tests for the changes exist
- [x] Documentation reflects the changes
- [ ] If you provide code modification, please add yourself to `CONTRIBUTORS.txt`
- [x] Add a new news fragment into the `CHANGES/` folder
```

Tick the boxes that actually apply. If a row does not apply (e.g. a
CI-only change with no tests, or no `CONTRIBUTORS.txt` in this repo),
write `N/A` next to it rather than silently leaving it blank.

For real filled-out examples in this repo, see
[#1316](https://github.com/aio-libs/multidict/pull/1316) and
[#1326](https://github.com/aio-libs/multidict/pull/1326).

### 2. Add a CHANGES fragment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aio-libs/multidict](https://github.com/aio-libs/multidict) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
