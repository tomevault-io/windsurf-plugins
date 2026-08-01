---
trigger: always_on
description: You are responsible for the **Processing** module of Retrace. Your job is to implement text extraction from captured frames using Vision framework OCR and the Accessibility API.
---

# PROCESSING Agent Instructions

You are responsible for the **Processing** module of Retrace. Your job is to implement text extraction from captured frames using Vision framework OCR and the Accessibility API.

**Status**: ✅ Vision OCR and Accessibility API fully implemented. **No audio transcription yet** (planned for future release).

## Your Directory

```
Processing/
├── ExtractMemoryInstrumentation.swift # Request-scoped extract residual/handoff instrumentation helper
├── ExtractRequestInstrumentation.swift # Request wrapper that drives extract-stage residual accounting
├── ProcessingManager.swift        # Main ProcessingProtocol implementation
├── FrameProcessingQueue.swift     # OCR pipeline orchestration + queue telemetry
├── URLExtractor.swift             # URL extraction from OCR text
├── OCR/
│   ├── VisionOCR.swift            # Vision framework OCR implementation
│   ├── VisionOCRHelpers.swift     # OCR output structs plus geometry/image helper methods
│   ├── VisionOCRInstrumentation.swift # OCR-local memory ledger runtime and tracker plumbing
│   ├── VisionOCRRequestConfig.swift # OCR request config type plus full-frame/region config builders
│   ├── VisionOCRResidualSupport.swift # OCR residual reset tables and reconciliation helpers
│   ├── FullFrameOCRCache.swift    # Cached full-frame OCR results for region re-OCR
│   ├── OCRTileCache.swift         # Tile cache support for region OCR
│   ├── RegionOCRMerger.swift      # Region OCR merge helpers
│   ├── RegionOCRResult.swift      # Region OCR result/stat models
│   ├── TileChangeDetector.swift   # Tile-based change detection
│   ├── TileGridConfig.swift       # Tile grid tuning
│   └── TileOCRProcessor.swift     # Tile OCR processing helpers
├── Accessibility/
│   ├── AccessibilityService.swift  # AccessibilityProtocol implementation
│   └── TextElementFilter.swift     # Filter relevant text elements
│
├── TextMerger/
│   └── TextMerger.swift            # Combine OCR + AX results
└── Tests/
    ├── ExtractRequestInstrumentationTests.swift # Region tail aggregation and coordinator helper coverage
    ├── InPageURLMetadataResolutionTests.swift # In-page URL metadata retry and rewrite scheduling regression coverage
    ├── OCRMemoryBackpressurePolicyTests.swift # OCR memory backpressure threshold/default coverage
    ├── PhraseLevelRedactionTests.swift        # Manual + automatic OCR phrase-level redaction coverage
    ├── RewriteRetryPolicyTests.swift          # Bounded automatic rewrite retry coverage
    ├── TestLogger.swift                       # Shared processing test logging helpers
    └── _future/
        ├── AccessibilityTests.swift
        └── VisionOCRTests.swift
```

## Protocols You Must Implement

### 1. `ProcessingProtocol` (from `Shared/Protocols/ProcessingProtocol.swift`)
- Text extraction (combined OCR + Accessibility)
- Configuration

### 2. `OCRProtocol` (from `Shared/Protocols/ProcessingProtocol.swift`)
- Vision framework text recognition

### 3. `AccessibilityProtocol` (from `Shared/Protocols/ProcessingProtocol.swift`)
- Permission checking
- Text extraction from AX tree

## Key Implementation Details

### 1. Vision Framework OCR

```swift
import Vision

struct VisionOCR: OCRProtocol {
    func recognizeText(
        imageData: Data,
        width: Int,
        height: Int,
        config: ProcessingConfig
    ) async throws -> [TextRegion] {
        // Create CGImage from raw data
        guard let cgImage = createCGImage(from: imageData, width: width, height: height) else {
            throw ProcessingError.imageConversionFailed
        }

        // Create request
        let request = VNRecognizeTextRequest()
        request.recognitionLevel = config.ocrAccuracyLevel == .accurate ? .accurate : .fast
        request.recognitionLanguages = config.recognitionLanguages
        request.usesLanguageCorrection = true

        // Perform recognition
        let handler = VNImageRequestHandler(cgImage: cgImage, options: [:])

        return try await withCheckedThrowingContinuation { continuation in
            do {
                try handler.perform([request])

                guard let observations = request.results else {
                    continuation.resume(returning: [])
                    return
                }

                let regions = observations.compactMap { observation -> TextRegion? in
                    guard observation.confidence >= config.minimumConfidence else { return nil }

                    let text = observation.topCandidates(1).first?.string ?? ""
                    let box = observation.boundingBox

                    return TextRegion(
                        text: text,
                        confidence: observation.confidence,
                        boundingBox: NormalizedRect(
                            x: box.origin.x,
                            y: box.origin.y,
                            width: box.width,
                            height: box.height
                        ),
                        source: .ocr
                    )
                }

                continuation.resume(returning: regions)
            } catch {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haseab/retrace](https://github.com/haseab/retrace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
