---
trigger: always_on
description: High-performance Go library for parsing EXIF metadata from JPEG, HEIC, AVIF, TIFF, DNG, CR2, CR3, CRW, NEF, ARW, RW2, and PNG. Targets parity with ExifTool as the reference implementation. Zero-allocation perceptual image hashing included.
---

# AGENTS.MD — imagemeta

## Project Overview

High-performance Go library for parsing EXIF metadata from JPEG, HEIC, AVIF, TIFF, DNG, CR2, CR3, CRW, NEF, ARW, RW2, and PNG. Targets parity with ExifTool as the reference implementation. Zero-allocation perceptual image hashing included.

## Core Principles

- **Idiomatic Go** — Use `gofmt`/`goimports` style. No JavaScript, Java, or Python patterns (no classes, no getters/setters, no builder patterns, no fluent interfaces).
- **Near-zero allocation** — Hot paths must avoid heap allocation. Use stack buffers (`[N]byte`), `sync.Pool`, and the `Acquire`/`Release` pattern. Always `b.ReportAllocs()` in benchmarks.
- **Functional options** — `ReaderOption func(*Reader)` pattern for optional config. Precompute lookup tables for option closures where possible.
- **Pooled resources** — Every pooled type has `Acquire*` / `Release*` functions. Always `defer Release*()` immediately after acquire.

## Project Structure

```
imagemeta/                  # Root module: github.com/evanoberholster/imagemeta
  imagemeta.go              # Public API: Decode(), DecodeCR3(), DecodeJPEG(), etc.
  imagetype/                # Image type detection (scan bytes, identify format)
  imagehash/                # Zero-alloc perceptual image hash (64/256 bit)
  preview/                  # Preview image extraction (CR3)
  meta/                     # Metadata model types (Exif, ExifHeader, etc.)
    exif/                   # Core EXIF IFD parser (main workhorse)
      tag/                  # Tag types, IFD types, value types
      makernote/            # MakerNote parsers
        canon/              # Canon makernote
        nikon/              # Nikon makernote
        sony/               # Sony makernote
        panasonic/          # Panasonic makernote
        apple/              # Apple makernote
    isobmff/                # ISO Base Media File Format (CR3, HEIC, AVIF)
    jpeg/                   # JPEG segment scanner
    png/                    # PNG chunk scanner
    xmp/                    # XMP XML parser (sax-based)
    logging/                # Zerolog-based logging mixin
    utils/                  # Byte order, bufio pool, limited reader
  cmd/                      # CLI tools
    compare_download_samples/  # Comparison tool for download_samples
  testImages/               # Small test images
  download_samples/         # Camera sample catalog (123 makes, NOT committed)
  golangci.yml              # Linter config
```

## Coding Conventions

### Imports
```
import (
    "stdlib"        // stdlib first, no blank line
    "io"
    "log/slog"
    "sync"

    "github.com/pkg/errors"              // third-party first group

    "github.com/evanoberholster/imagemeta/imagetype"  // internal second group
    "github.com/evanoberholster/imagemeta/meta"
)
```

### Error Handling
- **Sentinel errors**: Define in `var (...)` blocks using `errors.New` (stdlib), not `fmt.Errorf`:
  ```go
  var (
      ErrNoExif = meta.ErrNoExif
      ErrImageTypeNotFound = imagetype.ErrImageTypeNotFound
  )
  ```
- **Error wrapping**: Use `github.com/pkg/errors.Wrapf(err, "message")` with lowercase message, no trailing punctuation.
- **Error comparison**: Always use `errors.Is(err, ErrNoExif)`, never `==`.
- **Graceful degradation**: Skip invalid tags rather than aborting. Log non-fatal issues at warn level.
- **Conditional logging**: Always check level before creating log event: `if r.warnEnabled() { ... }`.

### Naming
- Exported types: PascalCase (`IFD0Tag`, `ExifIFDTags`, `Reader`)
- Unexported: camelCase (`state`, `eofReader`, `loggerMixin`)
- Constants: PascalCase exported (`ImageJPEG`), camelCase unexported
- Receiver names: single letter (`r *Reader`, `s *state`)
- Avoid stutter: `exif.Exif` not `exif.ExifReader`
- File names: short, single word where possible

### Pooling Pattern
```go
var parseReaderPool = sync.Pool{
    New: func() any { return &Reader{...} },
}

func AcquirePooledReader(l *slog.Logger) *Reader {
    r, ok := parseReaderPool.Get().(*Reader)
    if !ok || r == nil {
        r = &Reader{...}
    }
    r.resetDecodeState(true)
    return r
}

func ReleasePooledReader(r *Reader) {
    if r == nil { return }
    r.state.reset()
    statePool.Put(r.state)
    r.state = nil
    parseReaderPool.Put(r)
}
```

### Zero-Alloc Hot Path Guidelines
1. Prefer `[N]byte` arrays on stack over `make([]byte, N)`.
2. Use fixed-size tag queues (`[128]tag.Entry`) instead of dynamic slices.
3. Embedded values (≤4 bytes) read directly from IFD entry without allocation.
4. Returned byte slices reference parser buffers — document they are only valid until next read.
5. Use `sync.Pool` for everything: readers, states, bufio readers, pixel arrays.
6. Avoid `fmt.Sprintf` in hot paths; use structured slog fields.
7. Avoid `interface{}` allocations; use concrete types.
8. Use insertion sort for small fixed-size queues (avoids allocating `sort.Interface`).

### Code Generation
- `//go:generate msgp` for MessagePack serialization (`*_gen.go`)
- `//go:generate stringer -type=TypeName` for enum string methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evanoberholster/imagemeta](https://github.com/evanoberholster/imagemeta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
