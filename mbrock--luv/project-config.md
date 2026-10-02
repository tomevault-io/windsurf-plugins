---
trigger: always_on
description: Luv is a Common Lisp GPU workshop: a small WebGPU-shaped HAL with hand-built
---

# Working in Luv

Luv is a Common Lisp GPU workshop: a small WebGPU-shaped HAL with hand-built
Vulkan and native Metal 4 backends, an SDL3 canvas host, Lisp-defined
mathematical shaders, and a McCLIM GPU backend. It is inspectable machinery,
not a general-purpose engine.

Two game experiments exercise it. **Luvcraft** is the original procedural
block world, with streaming, simulation, persistence, and tools embedded in
the world. **Luft** is the current second-generation experiment: canonical
cubical topology, packed integer manifold-sheet meshes, and a playable McCLIM
atelier.

## Develop in the live image

Run `make` first in a new checkout or after pulling substantial changes. It
enters the checkout's Nix environment automatically and builds both
applications; slim agent setup builds the non-graphical `luft` system. Either
path also builds the `./sly` client and warms the FASL cache. Compiler detail
goes into `build/logs/`; the console stays small and names the slow files:

```text
$ make
000.4s  1/10  luft/luft.lisp
...
005.4s 10/10  luft/tests.lisp
;; Built :luft in 5.8s.
;; 10 files compiled, 10 loaded, 2 systems.

$ make test
parinfer: strict check passed.
....
Ran 4 tests in 0.2s
OK
LUFT: 22348 checks passed.
```

This makes the following Sly start mostly a load of already-compiled systems.
Test output is failure-focused: successful Parachute systems print one summary;
failures retain their descriptions and source locations.

Ordinary work happens in a durable SBCL image supervised by Swash. `./sly`
selects the image for this checkout and opens a fresh Slynk connection for
each command. Keep the application alive while inspecting and redefining its
code, classes, shaders, and tools.

```sh
./sly start                             # boot without opening a game
./sly play luft                         # current experiment
./sly play                              # Luvcraft instead
./sly status                            # image, game, and canvas health
./sly screenshot build/frame.png       # capture the GPU frame
./sly stop-playing                      # close the window, keep the Lisp
```

A cold start narrates compilation and ends with a useful summary. One real
start in this checkout said:

```text
;; Built (:luv-workbench) in 21s.
;; 214 files compiled, 250 loaded, 41 systems.
Lisp MKK067 (luv) is ready on 127.0.0.1:35435 (pid 936991).
```

Opening Luft returns the live viewer. The following commands then reported:

```text
$ ./sly status
Selected MKK067 (luv) for this checkout.
LUFT is playing: ./sly screenshot PNG; ./sly stop-playing closes it.
The canvas loop is healthy (waiting, 737 iterations, 10 frames).

$ ./sly screenshot build/frame.png
("/home/mbrock/luv/build/frame.png" 950 1188 :BGRA8-UNORM-SRGB)
```

The health comes from the canvas loop, not merely the Lisp connection. A frame
error parks drawing but leaves the window responsive; `./sly failures` shows
the retained conditions and backtraces, and `./sly resume` runs frames again
after the cause is fixed. Screenshots read the game's render target, not the
desktop.

## Explore before reading everything

The live roots are `luft.render:*viewer*` and `luvcraft:*session*`:

```sh
./sly eval '(type-of luft.render:*viewer*)'
./sly inspect 'luft.render:*viewer*'
./sly describe mesh-chunk --package LUFT
./sly apropos bevel --package LUFT
./sly edit mesh-chunk --package LUFT
./sly xref callers mesh-chunk --package LUFT
```

`describe` prints the real lambda list, derived type, documentation, and source
file. `xref` prints callers with file, line, and a source excerpt; asking for
callers of `mesh-chunk` leads directly into the chunked-meshing tests and
`luft/render/render.lisp`. `inspect` is interactive (`?` for commands, `q` to
leave).

An evaluation error prints a backtrace and Lisp restarts instead of killing
the image. It waits for a restart number or `a` to abort, so keep its stdin
visible rather than piping it through `head` or `tail`.

```sh
./sly load luft/render                  # hold frames during load, then fence
./sly systems                           # current, dirty, and unloaded systems
./sly system luft/render                # dependencies and pending ASDF actions
./sly stale                             # loaded systems with pending actions
```

The ASDF reports are read-only: `CURRENT` means a fresh `load-op` plan is
empty, `DIRTY` gives its action count, and `UNLOADED` means no successful load
is recorded in this image.

Swash also makes multiple images and worktrees explicit. `./sly list` shows
them all; create one with `./sly start --name experiment` and select it with
`./sly --lisp experiment ...`. An unqualified command refuses to guess when
several images match. `./sly log`, `restart`, and `stop` operate on the same
selected image.

## Environment and other tools

The dependencies live in the repository's `flake.nix`. `make`, `./sly`, and
the other repository launchers enter it automatically when needed. Use
`./env COMMAND` explicitly for an arbitrary command, or `./env` for an
interactive shell; `.envrc` activates the same environment through direnv.

Remote agents can use `./env --slim COMMAND` or
`nix develop .#slim`. It contains the complete Common Lisp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mbrock/luv](https://github.com/mbrock/luv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
