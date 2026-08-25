---
trigger: always_on
description: This repository creates a modified DOSBox Pure-based standalone Windows executable capable of running a complete DOS game from an archive embedded directly inside the executable.
---

# AGENTS.md

# DOSBox Pure Single-Executable Project Instructions

## Project Purpose

This repository creates a modified DOSBox Pure-based standalone Windows executable capable of running a complete DOS game from an archive embedded directly inside the executable.

The project must preserve the following core behavior:

- single distributable `.exe`
- no installation
- no extraction of the embedded game archive
- no temporary reconstruction of the archive on disk
- direct access to embedded game data from memory
- persistent save-game support through a writable overlay
- one-click startup
- support for disk images contained inside the embedded package

The detailed design and requirements are documented in:

```text
docs/architecture.md
docs/requirements.md
```

These documents are authoritative for project behavior.

---

# Repository Layout

Expected top-level structure:

```text
DOSBoxPureSingleExe/
│
├── AGENTS.md
├── README.md
│
├── docs/
│   ├── architecture.md
│   └── requirements.md
│
├── dosbox-pure-unleashed/
├── dosbox-pure/
├── ZillaLib/
│
├── packaging/
│
└── tools/
```

Roles:

```text
dosbox-pure-unleashed/
    Standalone Windows frontend and executable host.

dosbox-pure/
    DOS emulation core and primary location for archive/filesystem changes.

ZillaLib/
    Supporting frontend/runtime library used by DOSBox Pure Unleashed.

docs/
    Project architecture and requirements.

packaging/
    Future package resources and runtime templates.

tools/
    Future tooling such as makegame.exe.
```

---

# Primary Development Rule

Do not implement any solution that extracts the embedded game package to disk.

The following is forbidden:

```text
embedded archive
        |
        v
%TEMP%\game.zip
        |
        v
DOSBox Pure
```

The required architecture is:

```text
embedded archive
        |
        v
memory-backed random-access source
        |
        v
DOSBox Pure archive filesystem
```

This requirement applies even if the temporary file is:

- hidden
- deleted immediately after opening
- stored under AppData
- stored in a randomly named directory
- created only for compatibility

If a physical copy of the embedded game archive is created, the implementation does not satisfy the project requirements.

---

# Development Philosophy

Prefer small, isolated changes over large rewrites.

The project should reuse existing DOSBox Pure functionality wherever possible.

In particular, preserve and reuse:

- ZIP/DOSZ handling
- DOS filesystem behavior
- writable overlay logic
- disk-image support
- startup-script handling
- emulator behavior
- existing decompression paths

The goal is to replace or abstract only the backing source used to read the archive.

Avoid replacing major DOSBox Pure subsystems unless source analysis shows that this is unavoidable.

---

# Upstream Code

The following directories originate from upstream projects:

```text
dosbox-pure-unleashed/
dosbox-pure/
ZillaLib/
```

Avoid unnecessary formatting, restructuring, mass renaming, or cleanup in upstream code.

Changes should remain easy to compare against upstream.

When modifying upstream-derived code:

1. understand the existing call path first
2. make the smallest practical change
3. avoid unrelated refactoring
4. document why the modification is necessary
5. preserve normal behavior when practical

---

# Initial Technical Target

The first implementation milestone is NOT PE resource embedding.

First prove that DOSBox Pure can operate on a memory-backed archive.

Target test flow:

```text
game.zip on disk
        |
        v
load game.zip into memory
        |
        v
memory-backed random-access source
        |
        v
existing DOSBox Pure ZIP filesystem
        |
        v
game launches
```

This separates archive-source work from Windows resource work.

Only after this works should the source become:

```text
Windows PE resource
        |
        v
memory-backed random-access source
        |
        v
DOSBox Pure ZIP filesystem
```

---

# Investigation Before Modification

Before changing archive-loading code, trace the complete existing content-loading path.

Identify:

- how DOSBox Pure Unleashed receives the selected content filename
- how the filename enters DOSBox Pure
- where ZIP/DOSZ content is opened
- whether standard C file APIs or custom wrappers are used
- which functions require a full path
- how archive seeking is implemented
- how ZIP indexes are created
- how disk images inside archives are read
- how `.pure.zip` writable overlays are associated with the source archive
- how `DOSBOX.BAT` is detected and executed
- how save paths are generated

Record important findings in project documentation before making significant structural changes.

---

# Preferred Data Abstraction

Where practical, introduce a generic random-access archive source.

Conceptual API:

```cpp
class IDataSource
{
public:
    virtual ~IDataSource() = default;

    virtual uint64_t Size() const = 0;

    virtual size_t Read(
        uint64_t offset,
        void* destination,
        size_t bytes) = 0;
};
```

Expected implementations:

```text
FileDataSource
MemoryDataSource
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Buyukcaglar/DOSBoxPureStandalone](https://github.com/Buyukcaglar/DOSBoxPureStandalone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
