---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-format HDR image conversion tool. It decodes HDR-oriented
inputs to float32 linear scRGB-like data, then writes one of several HDR output
formats.

Supported input formats:

- JPEG XR: `.jxr`, `.wdp`, `.hdp`
- JPEG XL: `.jxl`
- OpenEXR: `.exr`
- AVIF: `.avif`
- HEIF/HEIC: `.heic`, `.heif`
- Ultra HDR JPEG: `.jpg`, `.jpeg`
- Radiance HDR: `.hdr`
- PNG: `.png`
- TIFF: `.tif`, `.tiff`

Supported output formats:

- Gainmap AVIF: default `.avif` output, using `avifgainmaputil_hdr.exe`
- JPEG XL HDR: `.jxl`
- Ultra HDR JPEG: `.jpg` / `.jpeg`
- Standard HDR AVIF: `.avif` with `--format avif`
- HEIF HDR: `.heic` / `.heif`

## Common Commands

```powershell
# Convert one file to gainmap AVIF.
python hdr2avif.py "C:\Users\77126\Videos\Forza Horizon 6\screenshot.jxr"

# Convert one file and infer output format from extension.
python hdr2avif.py input.jxr output.jxl
python hdr2avif.py input.jxr output.jpg
python hdr2avif.py input.jxr output.heic

# Write standard 10-bit PQ HDR AVIF instead of gainmap AVIF.
python hdr2avif.py input.jxr output.avif --format avif

# Write standard 10-bit PQ HDR HEIF.
python hdr2avif.py input.jxr output.heic --format heif

# Batch convert a directory.
python hdr2avif.py "C:\Users\77126\Videos\Forza Horizon 6" --output-dir .\output

# Batch convert to JPEG XL.
python hdr2avif.py "dir\" --output-dir .\jxl_out --format jxl

# Gainmap AVIF quality and headroom metadata mode.
python hdr2avif.py input.jxr -q 90 -s 4 --gainmap-headroom-mode source-peak

# Batch output naming.
python hdr2avif.py input1.jxr input2.jxr --output-dir .\output --name-pattern "HDR_{n}"

# JPEG XL lossless output.
python hdr2avif.py input.jxr output.jxl --lossless

# List supported formats.
python hdr2avif.py --list-formats
python hdr2avif.py --list-output-formats

# Inspect gain map metadata.
& .\tools\libavif\avifgainmaputil.exe printmetadata output.avif

# Install dependencies.
pip install -r requirements.txt
```

## Architecture

The core logic lives in the `src/hdr_transcoder` package. The old flat `src/`
modules (`cli.py`, `decoder.py`, `encoder.py`, `gainmap.py`, `processor.py`) are
now thin compatibility wrappers that re-export from `hdr_transcoder.*`.

```text
src/hdr_transcoder/
  config.py         -> Central constants, CICP codes, paths, timeouts
  color.py          -> Color-space matrices (sRGB↔BT.2020, gamma helpers)
  processor.py      -> SDR tone-mapping (prepare_base_sdr), PQ encoding (prepare_alternate_hdr)
  validation.py     -> Fidelity verification (peak/headroom checks, metadata validation)
  inspector.py      -> Image inspection, debug overlay generation, info JSON
  tools.py          -> Bundled tool paths, runtime environment checks
  tools_check.py    -> CLI entry for `python -m hdr_transcoder.tools_check`
  cli.py            -> CLI orchestration (convert_single, main, arg parsing)
  formats/
    __init__.py     -> Encoder dispatch (encode_output), format registry
    decoder.py      -> Multi-format decoder (decode_to_scrgb, probe_format)
    gainmap.py      -> Gainmap AVIF (avifgainmaputil_hdr.exe)
    jxl.py          -> JPEG XL (cjxl.exe), JXL_MODE_* constants
    avif.py         -> Standard AVIF HDR (imagecodecs)
    ultrahdr.py     -> Ultra HDR JPEG (imagecodecs)
    heif.py         -> HEIF HDR (pillow-heif)
hdr2avif.py         -> CLI entry: from hdr_transcoder.cli import main
jxr2avif.py         -> backward-compatible wrapper for hdr2avif.main()
```

### Data flow

```
Input → decoder.decode_to_scrgb() → float32 scRGB (H×W×3)
  ├─ Tier-1 formats (jxl/avif/heif/ultrahdr) → encoder.encode_output()
  └─ Gainmap AVIF → processor.prepare_base_sdr() + prepare_alternate_hdr()
                    → gainmap.encode_gainmap_avif()
```

### Fidelity model

| Mode | Default format | Description |
|------|---------------|-------------|
| `master` | lossless linear JXL | Archive/reprocessing (requires `--jxl-mode linear-srgb`) |
| `display` | Rec.2020 PQ JXL | Viewable HDR delivery |
| `compat` | Gainmap AVIF | Web-compatible with SDR fallback |

`--fidelity master` is the default. Non-JXL outputs under master require
`--allow-non-master`. Tier-1 formats (jxl, avif, heif, ultrahdr) are encoded
directly from scRGB; gainmap AVIF goes through the two-pass SDR+HDR pipeline.

### Bundled tools

Encode/decode relies on prebuilt executables in `tools/`:

- `tools/libjxl/` — cjxl.exe, djxl.exe, jxlinfo.exe (JPEG XL)
- `tools/libavif/` — avifgainmaputil.exe, avifgainmaputil_hdr.exe, avifdec.exe,
  avifenc.exe (AVIF)

These are required at runtime. `python -m hdr_transcoder.tools_check` reports
missing tools and dependency errors. `hdr_transcoder.tools` maps tool names to
absolute paths.

## Testing

```powershell
# Run quick tests (default — skips fidelity tests).
pytest

# Run all tests including slower end-to-end fidelity tests.
pytest -m ""

# Run only fidelity tests.
pytest -m fidelity

# Run a specific test file.
pytest tests/unit/test_tools_check.py

# Run tool invocation checks.
python -m hdr_transcoder.tools_check --invoke
```

Test markers (defined in pytest.ini):

| Marker | Description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yuuichu/hdr-image-transcoder](https://github.com/Yuuichu/hdr-image-transcoder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
