---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goal

Implement the DICOM standard as a TypeScript library. The `source-code/fo-dicom/` directory contains the **Fellow Oak DICOM** C# library (v5.2.5, targeting .NET Standard 2.0) as the reference implementation to port from.

## Repository Structure

```
dicom-ts/
|- source-code/fo-dicom/         # Reference C# implementation (READ-ONLY)
|  |- FO-DICOM.Core/            # Core DICOM classes to port
|  |- Platform/                 # Platform-specific rendering
|  |- Serialization/            # JSON serialization
|  `- Tests/                    # C# tests as reference cases
|- source-code/fo-dicom.Codecs/  # Codec-focused companion repo (READ-ONLY)
|  |- Codec/                    # C# codec manager + transfer syntax bindings
|  |- Native/                   # Native wrappers + third-party codec libs
|  `- Tests/                    # Acceptance fixtures for compressed syntaxes
`- src/                         # TypeScript source
```

## TypeScript Project Setup

No TypeScript project exists yet. When initializing:
- **Package manager: npm only** (not bun/pnpm/yarn)
- `npm init` — package.json with `"type": "module"`
- Target: ES2020, `"moduleResolution": "node16"`
- Output: dual CJS + ESM build using **two separate `tsconfig` files + plain `tsc`** (do NOT use tsup/esbuild/rollup — they carry native bindings that break on ARM/Dameng environments)
- devDependencies: **only 3 packages** — `typescript`, `vitest`, `@types/node`

## Dependency Policy — Minimize Third-Party Packages

**Runtime dependencies: zero npm packages.** Implement everything using Node.js built-ins:
- Deflate/zlib → `node:zlib`
- Charset decoding → Node.js built-in `TextDecoder` / `TextEncoder`
- UID generation → `node:crypto` (`randomUUID`, `randomBytes`)
- Network (DIMSE) → `node:net` / `node:tls`
- File I/O → `node:fs/promises`

**Codec strategy** (compression formats that cannot be trivially ported):
- RLE Lossless → pure TypeScript implementation
- JPEG Lossless Process 14 → pure TypeScript port of fo-dicom's `JpegLossless/` directory
- JPEG Lossy / JPEG2000 / HTJ2K → define `IDicomCodec` plugin interface only; users register their own codec via `TranscoderManager.register()` — no built-in implementation

Additional codec references:
- `source-code/fo-dicom/FO-DICOM.Core/Imaging/Codec/`: pure C# baseline behavior
- `source-code/fo-dicom.Codecs/Codec/`: transfer syntax coverage + codec registration patterns
- `source-code/fo-dicom.Codecs/Native/`: native interoperability behavior reference only (do not introduce native runtime dependency in `dicom-ts`)

Required codec transfer syntax coverage for Phase 10.5:
- JPEG Family
  - `1.2.840.10008.1.2.4.50` JPEG Baseline (Process 1), lossy 8-bit
  - `1.2.840.10008.1.2.4.51` JPEG Extended (Process 2 & 4), lossy 8/12-bit
  - `1.2.840.10008.1.2.4.57` JPEG Lossless (Process 14), all predictors
  - `1.2.840.10008.1.2.4.70` JPEG Lossless SV1 (Process 14, predictor 1)
- JPEG-LS Family
  - `1.2.840.10008.1.2.4.80` JPEG-LS Lossless
  - `1.2.840.10008.1.2.4.81` JPEG-LS Near-Lossless
- JPEG 2000 Family
  - `1.2.840.10008.1.2.4.90` JPEG 2000 Lossless
  - `1.2.840.10008.1.2.4.91` JPEG 2000 (lossy/lossless)
  - `1.2.840.10008.1.2.4.92` JPEG 2000 Multi-component Lossless
  - `1.2.840.10008.1.2.4.93` JPEG 2000 Multi-component

**Logger strategy:** define `IDicomLogger` interface; built-in console/null implementations only — no winston/pino.

This policy exists to ensure the library compiles and runs on ARM, domestic chips, and database-integrated environments (Dameng DB, Kingbase, etc.) without native module compilation issues.

## fo-dicom Reference Architecture

### Core Class Hierarchy (source-code/fo-dicom/FO-DICOM.Core/)

```
DicomFile                          → top-level container; wraps metadata + dataset
├── FileMetaInfo (DicomFileMetaInformation)
└── Dataset (DicomDataset)
    └── _items: SortedList<DicomTag, DicomItem>
        ├── DicomElement              → single data element with tag + VR + buffer
        │   ├── DicomStringElement    → string-valued elements
        │   └── DicomValueElement<T>  → numeric/binary-valued elements
        └── DicomSequence             → nested sequence of DicomDataset[]

DicomTag          → (group: ushort, element: ushort) pair; odd group = private
DicomVR           → value representation descriptor (LO, OB, SQ, UI, etc.)
DicomTransferSyntax → encoding scheme (ExplicitVRLittleEndian, JPEG, etc.)
DicomDictionary   → maps DicomTag → DicomDictionaryEntry (VR, name, VM)
```

### Reading Pipeline

```
DicomFile.Open(path)
  → DicomFileReader
    → DicomReader (IsExplicitVR, IsDeflated)
      → DicomReaderWorker  [observer pattern]
        → IDicomReaderObserver  [callbacks that build DicomDataset]
```

Key files:
- [FO-DICOM.Core/IO/Reader/DicomReader.cs](source-code/fo-dicom/FO-DICOM.Core/IO/Reader/DicomReader.cs)
- [FO-DICOM.Core/IO/Buffer/](source-code/fo-dicom/FO-DICOM.Core/IO/Buffer/) — IByteBuffer, LazyByteBuffer, BulkDataUriByteBuffer

### Network (DIMSE) Services

Base class: [FO-DICOM.Core/Network/DicomService.cs](source-code/fo-dicom/FO-DICOM.Core/Network/DicomService.cs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cocosip) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
