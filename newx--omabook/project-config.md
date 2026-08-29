---
trigger: always_on
description: Orientation for coding agents working in this repository. Humans want the
---

# CLAUDE.md

Orientation for coding agents working in this repository. Humans want the
[README](README.md); this file covers setup and the conventions that are not
obvious from the code.

**What this is.** omabook is a book library and reader for Omarchy — EPUB, PDF
and MOBI, with reading aloud and local AI. It was written in Rust with
[cxx-qt](https://github.com/KDAB/cxx-qt) and is being reimplemented here in
plain C++17 and Qt 6, so that it is built the same way as
[omawrite](https://github.com/omacom-io/omawrite), the Omarchy Markdown editor.
The Rust original lives at `~/Projects/omabook-rust` and is the reference for
behaviour; [SPEC.md](SPEC.md) is the reference for intent and
[TODO.md](TODO.md) tracks the port.

**omawrite is the style authority.** When this document and omawrite disagree,
omawrite wins and this document is wrong. There are exactly three deliberate
divergences, all because omabook is roughly ten times omawrite's size; each is
named and justified where it appears rather than left to be discovered.

---

## Setup

Two things are needed before a clone will run.

**1. Qt 6.** `qt6-base`, `qt6-declarative`, `qt6-webengine`, `qt6-webchannel`,
`qt6-imageformats`, `qt6-svg`, plus `poppler` for PDF covers and text. Arch
ships qmake as `qmake6`; `bin/build` resolves either name.

`qt6-imageformats` is not optional and is easy to miss: without it, WebP covers
— which some EPUBs use — decode to nothing and the book gets no cover, with no
error anywhere.

**2. foliate-js.** The reader engine is not vendored and is gitignored. Without
it the app builds and the library works, but opening a book renders nothing:

```bash
git clone https://github.com/johnfactotum/foliate-js.git assets/reader/foliate-js
```

## Build, test, run

```bash
./bin/build           # release, into build/
./bin/build --debug   # into build-debug/, faster to compile
./bin/test            # QtTest over src/core, and the boundary check
./build/omabook
```

`./bin/install` builds the Arch package through `makepkg -fsi`.

The library database lives at `~/.local/share/omabook/omabook.db`, with covers
beside it. Deleting that directory resets the app.

---

## Layout

```
omabook.pro           # the only build target; includes the two file lists
src/core/core.pri     # HEADERS/SOURCES for the core
src/core/             # library, import, search, speech, AI. Never QML.
src/app/app.pri       # HEADERS/SOURCES for the front end
src/app/              # QObjects exposed to QML, main.cpp
src/app/qml/          # every .qml, aliased flat into qrc:/
tests/tests.pro       # QtTest binary; compiles src/core directly
assets/reader/        # reader.html and foliate-js, loaded from disk over file://
pkgbuild/             # PKGBUILD, .desktop, icon
```

One `.pro`, one `make`, one installed binary — `/usr/bin/omabook` — exactly as
omawrite does it. The `.pri` files exist only so the `.pro` stays a page long;
they are file lists, not sub-projects. Do not reach for `TEMPLATE = subdirs` or
a static library: it buys no safety we do not already have and it breaks the
omawrite idiom where the test project compiles the real sources directly.

`src/core` knowing nothing about QML is the one architectural rule here. It is
what keeps a headless mode cheap to add later (SPEC §6.3), and a future
`omabook serve` is then a fifteen-line `.pro` that includes `core.pri`.

**The boundary is checked by the test build, not by convention.**
`tests/tests.pro` lists only `QT += sql network concurrent testlib` — no `qml`,
no `quick`, no `webenginequick`, no `multimedia` — so qmake never adds those
include paths and `#include <QQmlEngine>` in core code is a compile error rather
than a code review comment:

```
c.cpp:1:10: fatal error: QQmlEngine: No such file or directory
```

That is the direct translation of the Rust version's crate split, where
`omabook-core/Cargo.toml` simply did not list `cxx-qt`. Two caveats worth
knowing: the app build compiles the same files *with* those paths present, so
only `bin/test` catches a violation — run it; and the fully-qualified form
`#include <QtQml/QQmlEngine>` compiles anyway because `/usr/include/qt6` is
always on the path, so `bin/test` also greps for it.

**Where the boundary sits, exactly.** `src/core` may use QtCore, QtSql,
QtNetwork, QtConcurrent, and QtGui *for images only* — `QImage`, `QImageReader`,
`QColor`. It may not use QtQml, QtQuick, QtWebEngine, QtMultimedia, or QtWidgets.
A core class is a plain `QObject` — signals and slots are QtCore and are
encouraged — but nothing in core is written *for* QML: no `Q_INVOKABLE`, no
`qmlRegisterType`, no role enums, no `QVariantMap` shaped for a delegate. Those
belong to `src/app`.

**Divergence one: subdirectories.** omawrite puts every `.cpp`, `.h` and `.qml`
in one `src/`. At this size that would be ninety files in one folder, so core is
grouped by subsystem (`db/`, `models/`, `repo/`, `import/`, `ai/`, `tts/`). QML
stays in one directory, `src/app/qml/`, and every file is aliased to the `qrc:/`
root so components remain siblings and resolve each other through the implicit
directory import without any `import` statement.

---

## C++ conventions

These are read off omawrite's `backend.cpp` and `systemtheme.cpp`. Match them.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [newx/omabook](https://github.com/newx/omabook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
