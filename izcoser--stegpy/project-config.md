---
trigger: always_on
description: - This guide covers the Python project in this repository.
---

# AGENTS.md

## Scope
- This guide covers the Python project in this repository.
- Ignore `stegpy-rs/` unless the user explicitly asks about the Rust port. It is git-ignored and out of scope for normal work here.

## Project Summary
- `stegpy` is a small Python steganography CLI/library and FastAPI application for hiding text or file payloads inside images, WAV audio, and video.
- PNG/BMP/GIF/WebP/WAV use the original least-significant-bit path.
- JPEG/JPG now use a separate DCT-coefficient embedding path via `jpeglib`.
- MP4/M4V/MOV/MKV/WebM/AVI use a separate decoded-frame video DCT path via FFmpeg and are saved as MP4/H.264.
- The original package is centered around NumPy for byte-level manipulation, Pillow for image I/O, and `cryptography` for optional password-based encryption.
- The public CLI entry point is `stegpy=stegpy.steg:main`.
- The FastAPI application is exposed as `stegpy.web:app` and serves both `/api/*` endpoints and the static browser demo.

## Repository Layout
- `README.md`: user-facing overview and CLI examples.
- `pyproject.toml`: package metadata, dependencies, console script registration, and pytest configuration.
- `setup.py`: minimal setuptools compatibility shim.
- `stegpy/steg.py`: command-line interface.
- `stegpy/lsb.py`: main host-file abstraction plus encode/decode logic.
- `stegpy/video.py`: decoded-frame video DCT embedding and extraction through `ffmpeg`/`ffprobe`.
- `stegpy/crypt.py`: password-based encryption/decryption helpers using PBKDF2 + Fernet.
- `stegpy/web.py`: FastAPI upload, capacity, encode, and decode endpoints.
- `web-demo/`: static HTML/CSS/JavaScript frontend served by the FastAPI application.
- `tests/`: pytest-based characterization and end-to-end coverage.
- `images/`: example assets used in the README/demo.

## Core Flow
1. CLI parsing happens in `stegpy/steg.py`.
2. `lsb.HostElement` loads the host file and normalizes it into a NumPy array.
3. Payload bytes are wrapped in a small custom header:
   - magic number: `b"stegv3"`
   - payload length: 4 bytes, big-endian
   - embedded filename length: 1 byte
   - optional embedded filename bytes
   - raw payload bytes
4. If `-p/--password` is used, the formatted payload is encrypted before embedding.
5. `encode_message()` writes payload bits into the host array using 1, 2, or 4 LSBs per host byte.
6. JPEG hosts dispatch to a DCT-coefficient path that writes directly into quantized AC coefficients.
7. Video hosts dispatch to `stegpy.video`, which decodes frames, writes repeated payload bits into 8x8 luminance DCT coefficient relationships, and re-encodes MP4/H.264 output.
8. Extraction reverses the process, then optionally decrypts and either prints text or writes a recovered file prefixed with `_`.

## Web Application
- Run the local application from a source checkout with:
  - `uv run uvicorn stegpy.web:app --reload`
- The application exposes:
  - `GET /api/health`
  - `POST /api/capacity`
  - `POST /api/encode`
  - `POST /api/decode`
- `web-demo/` is mounted at `/`; opening `web-demo/index.html` directly does not provide the required `/api/*` backend.
- Image/audio host and file payload uploads are limited to 20 MB, local-web video hosts are limited to 5 MB, and text messages are limited to 1 MB.
- `/api/capacity` reports raw usable payload bytes after accounting for the payload header, embedded filename, and optional Fernet encryption expansion.
- The web API accepts only PNG, BMP, GIF, WebP, WAV, JPG, JPEG, MP4, M4V, MOV, MKV, WebM, and AVI hosts; unlike the CLI/library path, it does not accept arbitrary Pillow-readable image formats for conversion.
- Video hosts are allowed only for local web requests (`localhost`, `127.0.0.1`, `::1`, and `*.localhost`). The hosted public demo rejects video hosts before saving uploads because FFmpeg processing can saturate the VPS for minutes.
- Uploads and generated files are processed in temporary directories and removed by response background tasks.
- `web-demo/` is installed as wheel data under the environment prefix; `stegpy.web` falls back to that location outside a source checkout.
- Host parsing, capacity calculation, encoding, and decoding are dispatched through Starlette's thread pool so CPU-bound image/audio work does not block the event loop.

## File Format Handling
- Images:
  - Non-GIF image hosts are opened with Pillow and converted to `RGB` if needed.
  - Supported save targets are effectively lossless image formats plus format-conversion to PNG when the original extension is unsupported.
- JPEG:
  - JPEG hosts are loaded and saved through `jpeglib`, not Pillow pixel arrays.
  - Payload bits live in quantized AC DCT coefficients rather than pixel bytes.
- GIF:
  - Each frame is loaded into a NumPy array and palettes/duration are preserved separately.
  - Save reconstructs the animation frame-by-frame.
- WAV:
  - The code reads raw bytes with NumPy and treats the first `10000` bytes as "header" and the remainder as mutable payload area.
  - This is a project-specific simplification, not a general WAV parser.
- Video:
  - Video support requires `ffmpeg` and `ffprobe` on PATH.
  - Input containers are decoded to raw RGB frames, embedded through a DCT signal on luminance, and re-encoded as MP4/H.264 with `libx264`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [izcoser/stegpy](https://github.com/izcoser/stegpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
