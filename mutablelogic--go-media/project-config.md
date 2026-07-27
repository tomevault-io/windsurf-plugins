---
trigger: always_on
description: This Go module provides CGO bindings to several media C libraries, layered consistently:
---

# go-media AI Agent Instructions

## Architecture Overview

This Go module provides CGO bindings to several media C libraries, layered consistently:

1. **sys/\*** - low-level CGO bindings, mirroring each C library's API 1:1
   - `sys/ffmpeg80/` (default), plus `sys/ffmpeg71/`, `sys/ffmpeg61/` - FFmpeg bindings, selected via `SYS_VERSION` in the Makefile
   - `sys/libheif/`, `sys/libraw/`, `sys/libexif/`, `sys/chromaprint/`, `sys/dvb/`
2. **pkg/\*** - high-level, idiomatic Go APIs per library
   - `pkg/ffmpeg/` - Reader, Decoder, Encoder, Resampler, Frame abstractions
   - `pkg/heif/`, `pkg/raw/`, `pkg/exif/` - `Open`/`Read`/`Parse` returning a closable value; RAW and HEIF also implement `image.Image` and register with the stdlib `image` package
   - `pkg/xmp/` - XMP document read/write, used to serialize extracted metadata
   - `pkg/sdl/` - SDL2 video/audio player (Context, FrameLoop, Player). **Not currently wired into the `gomedia` CLI** - it's an independent, tested library; treat it as such rather than assuming there's a `play` command.
   - `pkg/chromaprint/` - audio fingerprinting
3. **metadata/** - format-agnostic metadata extraction, independent of `pkg/ffmpeg`
   - `metadata/metadata.go` defines `AddHandler(re *regexp.Regexp, fn HandlerFunc, namespaces ...string)`; each format package (`metadata/image`, `metadata/audio`, `metadata/video`, `metadata/application`) registers itself in an `init()` against a content-type regex and the metadata namespaces it can produce (`tiff`, `exif`, `image`, `dc`, `artwork`, ...)
   - `metadata.GetMetadata(ctx, r, contentType, filter)` runs every matching handler concurrently and merges results; a returned error is the *joined* errors of failing handlers and should be treated as a warning, not a reason to discard metadata that did come back
   - `filter` follows a `namespace:name` convention, e.g. `"exif:"` (whole namespace), `"DateTimeOriginal"` (bare name, any namespace), `"exif:DateTimeOriginal"` (exact), `"artwork:thumbnail"` (embedded preview image)
4. **gomedia/** - the application layer that the CLI/server binary is built from
   - `gomedia/schema/` - request/response types with `kong` CLI tags (the public-facing API surface; distinct from the lower-level `pkg/ffmpeg/schema/` types used internally)
   - `gomedia/manager/` - `manager.Media` orchestrates `pkg/ffmpeg`, `metadata/`, `pkg/chromaprint`, `pkg/xmp` behind the `gomedia/schema` types (`Probe`, `GetMetadata`, `SegmentAudio`, `ListCodecs`, etc.)
   - `gomedia/cmd/` - `kong`-based CLI command definitions (`CLICommands` embeds `MetadataCLICommands`, `CapabilitiesCLICommands`, `EncodingCLICommands`); each `Run` method calls `c.WithManager(ctx, func(m *manager.Media) error {...})`
5. **cmd/gomedia/** - thin `main()` that wraps `gomedia/cmd.CLI{}` via `go-server`'s `cmd.Main(...)`. There is no bespoke input-format-sniffing or SDL-wiring logic here anymore - look in `gomedia/manager` and `gomedia/cmd` instead.

An older Task Manager pattern under `pkg/ffmpeg/task/` and an SDL-playback `main.go` are referenced by some stale docs/README content but no longer exist on this branch (that code now lives archived under `_old/`, if present). Don't assume `pkg/ffmpeg/task` exists - the equivalent responsibility now lives in `gomedia/manager`.

## Critical Build Workflow

**First build ALWAYS requires the full `make`** - it builds several C dependencies as static libraries before compiling Go:
```bash
make              # Builds ffmpeg, chromaprint, libexif, libraw, libheif, then all cmd/* binaries
```

Source versions are pinned in the Makefile (check there for current values - they change):
```
FFMPEG_VERSION, CHROMAPRINT_VERSION, LIBEXIF_VERSION, LIBRAW_VERSION, LIBHEIF_VERSION
```

**Subsequent builds** can target a single command once the C deps are already installed under `build/install`:
```bash
make cmd/gomedia  # Rebuilds Go only, IF ffmpeg/chromaprint/libexif/libraw/libheif are already built
```

The Makefile sets CGO env vars for every build/test target (`CGO_ENV`), and **these differ by OS**:
```bash
# common to both
PKG_CONFIG_PATH="$(realpath build/install)/lib/pkgconfig"
CGO_LDFLAGS_ALLOW="-(W|D).*"
# darwin adds -Wl,-no_warn_duplicate_libraries to CGO_LDFLAGS; linux does not
CGO_LDFLAGS="-lstdc++ [-Wl,-no_warn_duplicate_libraries on darwin]"
```
Never run `go build`/`go test` directly without these vars (see Makefile's `CGO_ENV`) or you'll get pkg-config/link errors. If you must run `go test` outside `make`, set `PKG_CONFIG_PATH` to `$(realpath build/install)/lib/pkgconfig` at minimum.

### libheif codec selection is dependency-driven, not hardcoded

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mutablelogic/go-media](https://github.com/mutablelogic/go-media) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
