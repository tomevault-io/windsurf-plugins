---
trigger: always_on
description: Guidance for AI assistants and contributors working in this repository.
---

# CLAUDE.md

Guidance for AI assistants and contributors working in this repository.

## Project

**pe-builder** is an open-source, cross-platform tool for building bootable Windows
PE environments from a user-supplied Windows installation source. It is a
clean-room, Linux-first reimplementation of the ideas behind the discontinued
*Bart's PE Builder*, aiming for compatibility with that tool's plugin format. See
[`README.md`](README.md) for the project overview.

## Repository layout

The build **engine** is a set of importable Go packages; `cmd/pebuild` is a thin CLI
over it. `pewalk` is a second, standalone tool (library + CLI) that inspects a source's
PE dependencies.

- `cmd/pebuild/` — the command-line frontend (cobra). Besides building, it exposes the
  engine's readers as inspection commands: `source probe|extract|ls|dirs|inf` reads a
  Windows installation source (identity, a file pulled out and decompressed from
  whichever form the media stores it in, the declared file list with each file's storage
  form and output path, the directory-ID table, and an INF section as the build's own
  parser sees it), and `hive dump` prints a key or value from a registry hive in a
  source, a built PE, or a bare hive file. `run` boots an image `build` already produced —
  named directly, or through the manifest that wrote it — in QEMU, reading the image's own
  architecture to pick the emulator and machine; it deliberately does not build, and is a
  plain launcher. Its `--serial` attaches COM1 to a QEMU chardev, which is what an image built
  with the `serialdebug` plugin needs to be debuggable — that plugin turns the kernel debugger
  on, and a debug image booted with no debugger attached suppresses its own output rather than
  producing any. `verify` answers the question an
  image on disk otherwise cannot — what was this built from, and is that still what this
  manifest and this pebuild describe? — by reading the receipt the image carries and
  recomputing the inputs it records from
  the current binary, manifest and plugin directories, naming which of engine, manifest,
  source, plugin set or options differs. It is a provenance report over a build's *inputs*,
  not evidence that the binary in hand would reproduce the image, and its verdict line names
  what it compared so the boundary is visible where the answer is. With no manifest to be
  found it just prints the
  receipt, so it stays useful away from the tree that built the image. It exits non-zero
  on a mismatch, and keeps three outcomes apart that are easy to conflate: a mismatch, an
  input that could not be checked, and an image carrying no receipt at all.
- `cmd/pewalk/` — the `pewalk` command: an "ldd for PE" that computes and renders the
  import closure of a Windows binary against a source (cobra).
- Engine packages, roughly in pipeline order:
  - `manifest/` — the HCL build manifest (load + validate). Its `layout` switch picks how
    the ISO is assembled: `classic` (the default) masters the file tree directly; `wim`
    packs the tree into a compressed `.wim` behind a fully distributable grub4dos chain that
    maps a small boot-core image (`coreimg`) into RAM as a CD and boots it with an unmodified
    loader, so it ships no Microsoft boot binary (see `build/` and the `wim-cd-boot` plugin
    below). The `wim` layout requires an ISO target.
  - `source/` — read a Windows installation source (directory or ISO) and probe its
    identity (build number, architecture, flavor). It also owns the source's *areas*:
    a source keeps its files in two top-level folders — the OS payload and the
    real-mode boot chain — which are the same `I386` folder on x86 media and split on
    amd64 media, where the payload moves to `AMD64` and the boot chain (being real
    mode) stays in `I386`. `Identity.Roots` resolves the pair, and every stage that
    reads from or writes under a source folder resolves it through that rather than
    naming one, so they cannot disagree.
  - `compress/` — decompress the source's MSCF cabinets (`.??_` files, `DRIVER.CAB`,
    the `ASMS*.CAB` assembly cabinets). A cabinet declares its compression per folder,
    so one may mix codecs: LZX covers essentially everything on x86 media, while amd64
    media packs the side-by-side assemblies with MSZIP (deflate per block, each block
    decoded against its predecessor's output as a preset dictionary).
  - `inf/` — a Windows setupapi-compatible `.inf` parser, and the PE adaptation of a network
    component's INF: a stock one applies security descriptors a PE has no API to apply and
    starts co-services a minimal PE does not run, so a staged copy has both removed. It
    rewrites the *user's* file at build time rather than shipping an adapted copy of
    Microsoft's, which is what keeps it clean-room.
  - `layout/` — resolve the source's `txtsetup.sif` / `layout.inf` (file → target
    directory, on-disk name, boot-driver lists, and the source's locale from the
    `[nls]` section). It also settles whether the source is a workstation or a server
    product — the one thing separating XP Professional x64 Edition from Server 2003 x64,
    which are otherwise the same build number and architecture — from the product type

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nradchenko/pe-builder](https://github.com/nradchenko/pe-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
