---
trigger: always_on
description: Guidance for AI assistants (and humans) working in this repository. Read this
---

# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository. Read this
first — it captures the project's purpose, layout, build system, architecture, and
conventions so you can be productive immediately.

---

## 0. HARD RULE — never edit code, write a proposal instead

**The owner of this repo writes all the code. AI assistants do not.**

Whenever a request would require *any* change to the source tree — new files,
edits, deletions, renames, `CMakeLists.txt` entries, config tweaks — **do not
touch a single file.** Instead:

1. Write the complete proposal to a markdown file under **`notes.dev/`**
   (git-ignored via the existing `*.dev*` rule), named after the task, e.g.
   `notes.dev/cookie_storage_plan.md`.
2. In that file, give the **full** intended content — whole functions, whole
   headers, exact diffs, exact `CMakeLists.txt` lines — not a summary. The owner
   copies from it and types the real change themselves, so anything left vague is
   work they have to redo.
3. Reply in chat with the file path and a short summary. Nothing else.

Applies to:

- All of `include/`, `src/`, `scripts/`, `assets/`, `.github/`, `CMakeLists.txt`,
  `CMakePresets.json`, `vcpkg.json`, `.clang-format`, `.gitignore`, and any other
  tracked file.
- Scaffolding scripts (`scripts/new_class.*`) — **do not run them**, they create
  files and edit `CMakeLists.txt`. Write out what they *would* generate instead.
- Refactors, bug fixes, and one-line changes alike. "It's tiny" is not an
  exception.
- Git-mutating commands: no `commit`, `add`, `checkout`, `restore`, `stash`,
  `rm`, `mv`.

Still allowed without asking:

- Reading, searching, and `git` inspection (`status`, `diff`, `log`, `show`).
- Configuring, building, and running the app to reproduce or verify behaviour.
- Writing/updating files inside `notes.dev/` (and other `*.dev*` paths).
- Editing this `CLAUDE.md` when explicitly asked to record a rule.

If a request seems to *require* editing to be useful, it doesn't — write the
proposal. The only way this rule is bypassed is the owner saying so explicitly,
for that one request.

---

## 1. HARD RULE — prefer Boost for new code

**Boost is a first-class dependency here. When new code needs a utility that
Boost provides, use Boost rather than hand-rolling it or reaching for a
hand-written loop.**

This applies to proposals too: a plan written into `notes.dev/` must already use
Boost where Boost fits — don't propose a manual `std::string::find` / `substr`
parser and leave the Boost version as a footnote.

Where it matters most today:

| Need | Use | Not |
|------|-----|-----|
| trim / case-insensitive compare / starts-with / split / replace / join | `boost/algorithm/string.hpp` (`boost::trim`, `boost::iequals`, `boost::istarts_with`, `boost::split`, `boost::replace_all`) | hand-written `find`/`substr` loops, `std::transform(::tolower)` |
| substring / delimiter search returning a range | `boost::algorithm::find_first` / `find_last` (`boost/algorithm/string/find.hpp`) | manual index arithmetic around `npos` |
| clamping | `boost::algorithm::clamp` | hand-written `min(max(...))` |
| any_of / all_of over a range with a classifier | `boost/algorithm/cxx11/any_of.hpp` + `boost::is_any_of`, `boost::is_space` | raw loops |
| filter/transform pipelines into a container | Boost.Range adaptors (`boost::adaptors::filtered`, `::transformed`) + `boost::push_back` | manual push_back loops |
| small containers that usually fit on the stack | `boost::container::small_vector<T, N>` | `std::vector` when N is known and small |

Existing precedent to follow — read these before writing new string/range code:

- `src/av_ui/input_autocomplete_ui.cpp` — the heaviest Boost user (algorithm,
  range adaptors, `small_vector`).
- `src/av_net/network_manager.cpp` — header/cookie parsing built on
  `boost::algorithm::find_first`, `trim`, `iequals`, `starts_with`.
- `src/av_ui/search_view_ui.cpp` — query tokenizing via `boost::split` + `trim`.

Rules of engagement:

- **Header-only Boost only.** `CMakeLists.txt` links `Boost::headers`; nothing
  links a compiled Boost library. If a feature needs a compiled Boost component
  (Boost.Filesystem, Boost.Thread, …), do **not** silently add it — call it out
  in the proposal, since it changes `vcpkg.json`, the link line, and the static
  Windows build.
- **Adding a Boost sub-library means editing `vcpkg.json`.** Today only
  `boost-algorithm` and `boost-container` are declared. A new `boost/…` include
  from a different sub-library needs its vcpkg port added *and* a reconfigure
  (`cmake --preset <name>`) — flag both in the proposal.
- **Include the narrow header** (`boost/algorithm/string/trim.hpp`) over the
  umbrella `boost/algorithm/string.hpp` when only one facility is used; both
  patterns exist in the tree, narrow is preferred for new code.
- **Don't replace working `std` code with Boost for its own sake.** The rule is
  about *new* code (and code you're already rewriting). `std::string_view`,
  `std::optional`, `std::filesystem`, `std::async`/`std::future`, and
  `std::unique_ptr`/`shared_ptr` stay as they are — Boost is not a substitute for
  the standard library where the standard library is already idiomatic here.

---

## 2. What this project is


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rennn0/arvis](https://github.com/Rennn0/arvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
