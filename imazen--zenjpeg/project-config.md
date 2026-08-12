---
trigger: always_on
description: 1. The jpegli encoder source is in @lib/jpegli /lib/jpegli
---


# Rules for porting the C++ jpegli encoder algorithms to the Rust jpeg-encoder crate

1. The jpegli encoder source is in @lib/jpegli /lib/jpegli
2. jpeg-encoder is in @jpeg-encoder /jpeg-encoder
3. We target stable Rust, and keep any unsafe code (like SIMD abstractions) simple; study jpeg-encoder and follow those patterns. 
4. Before porting a C++ component, we examine all the headers it references and build a list of all the functions it actually depends on, and add that info as comments in the C++ header.
5. We work methodically, search for a replacement in jpeg-encoder or create one, and try to create idomatic but performant and correct solutions. 
6. We add new rules when we glean insight about jpegli, its structure, organization
7. We add rules whenever we establish a mapping from a C++ component to a rust component, including function signatures.
8. We always port tests and run them regularly.

## Rule 9: Dependencies of lib/jpegli/encode.cc

The main encoder implementation in `lib/jpegli/encode.cc` depends on the following headers:

*   **C API:** `lib/jpegli/encode.h`
*   **Standard Libraries:** `<algorithm>`, `<cstddef>`, `<cstdint>`, `<cstring>`, `<vector>`
*   **Jpegli Base:** `lib/base/types.h`
*   **Jpegli Common:** `lib/jpegli/common.h`, `lib/jpegli/common_internal.h`, `lib/jpegli/types.h`, `lib/jpegli/error.h`, `lib/jpegli/memory_manager.h`, `lib/jpegli/simd.h`
*   **Jpegli Encoding Stages:**
    *   `lib/jpegli/input.h`
    *   `lib/jpegli/color_transform.h`
    *   `lib/jpegli/downsample.h`
    *   `lib/jpegli/adaptive_quantization.h`
    *   `lib/jpegli/quant.h`
    *   `lib/jpegli/entropy_coding.h`
    *   `lib/jpegli/huffman.h`
    *   `lib/jpegli/bitstream.h`, `lib/jpegli/bit_writer.h`
    *   `lib/jpegli/encode_streaming.h`, `lib/jpegli/encode_finish.h`
*   **Internal Helpers:** `lib/jpegli/encode_internal.h`

These represent the primary modules involved in the JPEG encoding process within jpegli.


Based on our analysis of the `jpegli` encoder source code and its API (`encode.h`, `encode.cc`), here are some key algorithmic differences compared to a standard `libjpeg-turbo` implementation:

1.  **Adaptive Quantization:** Jpegli implements and enables *adaptive quantization* by default (`jpegli_enable_adaptive_quantization`, `ComputeAdaptiveQuantField`). This means it analyzes local image features (like edges and textures) and adjusts the quantization strength accordingly, aiming to preserve detail where it's visually important and save bits where it's not. Standard libjpeg uses non-adaptive quantization unless specific extensions (like Trellis quantization, often slower) are enabled.
2.  **Psychovisually Tuned Quantization Tables & Distance Metric:** Jpegli uses different default quantization tables than the standard Annex K tables used by libjpeg-turbo. These tables are likely derived from psychovisual modeling (related to the Butteraugli metric). Instead of just a `quality` factor (0-100), jpegli allows setting a target *Butteraugli distance* (`jpegli_set_distance`) which provides a more perceptually uniform measure of image quality/difference. While `jpegli_set_quality` exists for compatibility, it maps to an underlying distance. Libjpeg-turbo's quality setting directly scales the standard tables. Jpegli *can* use the standard tables if requested (`jpegli_use_standard_quant_tables`).
3.  **XYB Color Space Option:** Jpegli offers the option to use the perceptually optimized XYB color space (`jpegli_set_xyb_mode`) derived from JPEG XL. This can lead to better compression efficiency compared to the traditional YCbCr color space used by default in libjpeg-turbo for color images.
4.  **Default DCT Method:** Jpegli seems to default to a floating-point DCT (`cinfo->dct_method = JDCT_FLOAT;` in `InitializeCompressParams`). While libjpeg-turbo *supports* float DCT, its default and often faster options are integer DCTs (`JDCT_ISLOW`, `JDCT_IFAST`).
5.  **Different Defaults:** Jpegli uses different defaults for parameters like progressive scan scripts (`jpegli_set_progressive_level`, `SetDefaultScanScript`) aiming for potentially better compression or perceived quality out-of-the-box.

**In summary:** While `jpegli` maintains API compatibility with `libjpeg-turbo`, its internal algorithms are significantly enhanced with techniques focused on psychovisual optimization. It leverages adaptive quantization, perceptually derived quantization tables (Butteraugli-based), and the XYB color space to achieve higher quality compression for a given file size compared to standard JPEG encoders like `libjpeg-turbo`, often at comparable or slightly slower speeds depending on the features used.


Okay, based on the analysis of the `jpeg-encoder` crate's source code (`lib.rs`, `encoder.rs`, and other modules), here's an outline of its structure and an assessment of what can be reused for porting `jpegli`:

**Structure of `jpeg-encoder`:**

1.  **Main Entry Point (`lib.rs`, `encoder.rs`):**
    *   Defines the public API (`Encoder`, `ColorType`, `SamplingFactor`, etc.).
    *   The `Encoder` struct acts as the central orchestrator, holding configuration (quality, sampling, tables, progressive settings, writer) and managing the encoding process.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imazen/zenjpeg](https://github.com/imazen/zenjpeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
