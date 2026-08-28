---
trigger: always_on
description: generates its source archives on demand and they are not byte-stable across git
---

# AGENTS.md

Working notes for anyone — human or agent — changing this repository.

**What each file answers.** The user-facing meaning of a setting — what it does,
what values it takes, what it defaults to — is written once, in
`amberedit.cfg.example`. This file says only what the code has to guarantee
about it: where it is read, where it is applied, and what breaks if that changes.

**Every document here describes the project as it stands.** README.md, INSTALL.md,
`amberedit.cfg.example`, this file and the comments in the code all say what is
true now, in the present tense. What was tried and taken out, what a setting used
to be called, which approach lost — none of it is recorded: it is one more thing
to keep in step with the code, and a reader who has to work out which paragraphs
are still true has been given work rather than answers. When something changes,
rewrite the sentence rather than adding "previously…" beside it, and delete what
has stopped being true. History belongs in a changelog, and there is no such file
in this repository yet; if one is added, it is the single exception to this rule
and nothing leaks back into the documents above.

The rule is about *narrative*, not about reasons. A rule that keeps the next
change from undoing a deliberate decision stays — written as what holds now and
why, never as the story of what happened: "horizontal scrolling is deliberately
unhandled, because nothing the terminal reports carries an event's phase" and not
"we tried swipe-to-next-message and removed it".

Contents: [What AmberEdit is](#what-amberedit-is) ·
[Build and test](#build-and-test) · [Packaging](#packaging) ·
[Layering](#layering) ·
[Code conventions](#code-conventions) · [Domain notes](#domain-notes) ·
[The message base drivers](#the-message-base-drivers) ·
[The nodelist](#the-nodelist) · [The echolist](#the-echolist) ·
[Commands and the keyboard](#commands-and-the-keyboard) ·
[Reference material in the tree](#reference-material-in-the-tree) ·
[Current scope](#current-scope)

## What AmberEdit is

A TUI Fidonet mail editor. It reads message bases (Squish, JAM, Fido `*.msg`)
on top of an existing tosser configuration and does not duplicate its settings.
It reads and writes message bases, but it is not a mailer and not a tosser: what
it writes goes out only when a tosser carries it.

## Build and test

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j
./build/bin/amberedit_tests          # or: ctest --test-dir build
```

Both must be clean before anything is called done, and the whole thing must
build from a fresh clone with no network. The build produces one binary; the
external inputs are the wide ncurses, iconv, zlib — for the zipped nodelists and
echolists AmberEdit unpacks itself — tl::expected, which every fallible operation
answers with, and doctest for the tests, all of them found on the system and none
of them fetched.

It also produces the message catalogs, one
`build/locale/<lang>/LC_MESSAGES/amberedit.mo` per `po/*.po`, where `msgfmt` is on
the system. `msgfmt` is not required and the build warns rather than stopping: a
build without it is a build with the English the source is written in, which is
every string the program has. gettext's *runtime* — `libintl`, part of libc on
glibc and a library of its own on macOS — is required, since it is what reads a
catalog.

Test cases carry their tags inside the name — `TEST_CASE("... [nodelist][ui]")` —
because doctest's `TEST_CASE` takes a name and nothing else. They are filtered
with `-tc`, which matches the whole name by wildcard:
`./build/bin/amberedit_tests -tc="*[squish]*"`.

The code is C++17 and wants GCC 8, Clang 7 or Apple Clang 11 at the least;
`CMakeLists.txt` refuses anything older by name, because an older compiler
accepts `-std=c++17` well enough for CMake to call `CMAKE_CXX_STANDARD 17`
satisfied and then buries the build in errors from inside `<bits/>`.

**That floor is the point of the standard, not an accident of it.** RHEL 8 and
its rebuilds must build AmberEdit out of the box, with the stock GCC 8.5 and the
CMake 3.20–3.26 the release ships — no `gcc-toolset`, no newer CMake. Worth
checking after anything that touches the build or a header, because a Mac's
Clang accepts most of C++20 under `-std=c++17` with only a warning:

```bash
docker run --rm -v "$PWD":/src:ro rockylinux:8 sh -c '
  dnf -y install epel-release &&
  dnf -y install gcc-c++ cmake make ncurses-devel zlib-devel \
                 expected-devel doctest-devel &&
  cp -a /src /work && rm -rf /work/build &&
  cmake -S /work -B /out && cmake --build /out -j"$(nproc)" &&
  cd /out && ctest --output-on-failure'
```

EPEL comes first and on its own line because two of the packages behind it are
there and nowhere else on RHEL 8: `expected-devel` and `doctest-devel`. Without
it `cmake` stops at the tl::expected check, which reads as a broken tree rather
than as a missing repository.

The copy is not incidental: one test opens `testdata/msgbase/charsets` in the
source tree itself, so a read-only `/src` fails there and nowhere else. Short of
a container, `-Werror=c++20-extensions` on a Clang build catches the language
half of the same thing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jegornet/amberedit](https://github.com/jegornet/amberedit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
