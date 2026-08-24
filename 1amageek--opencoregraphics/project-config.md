---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenCoreGraphics targets full CoreGraphics API compatibility for WebAssembly (WASM) environments. Current completion is measured per API and rendering path.

### Core Principle: Full Compatibility

**The target API must be 100% compatible with CoreGraphics.** This means:
- Identical type names, method signatures, and property names
- Implemented behavior and semantics must be independently validated against CoreGraphics
- Compatibility claims must identify the exercised surface and evidence

### How `canImport` Works

Users of this library will write code like:

```swift
#if canImport(CoreGraphics)
import CoreGraphics
#else
import OpenCoreGraphics
#endif

// This code works in both environments
let rect = CGRect(x: 0, y: 0, width: 100, height: 100)
```

- **When CoreGraphics is available** (iOS, macOS, etc.): Users import CoreGraphics directly
- **When CoreGraphics is NOT available** (WASM): Users import OpenCoreGraphics, which provides identical APIs

This library exists so that cross-platform Swift code can use CoreGraphics APIs even in WASM environments where Apple's CoreGraphics is not available.

## Build Commands

```bash
# Build the package (macOS)
swift build

# Run focused tests (macOS) with a process timeout
perl -e 'alarm 30; exec @ARGV' -- \
  xcodebuild test -scheme OpenCoreGraphics -destination 'platform=macOS' \
  -only-testing:OpenCoreGraphicsTests

# Build for WASM
TOOLCHAINS=org.swift.64202607171a xcrun swift build \
  --swift-sdk swift-6.4.x-DEVELOPMENT-SNAPSHOT-2026-07-17-a_wasm
```

## Architecture

### 設計原則: CoreGraphicsと完全に同じ使い方

ユーザーはネイティブでもWASMでも**完全に同じコード**を書きます。初期化関数やレンダラー設定は不要です。

```swift
#if canImport(CoreGraphics)
import CoreGraphics
#else
import OpenCoreGraphics
#endif

// これだけ。初期化関数は不要。CoreGraphicsと完全に同じAPI。
let context = CGContext(
    data: nil,
    width: 800,
    height: 600,
    bitsPerComponent: 8,
    bytesPerRow: 0,
    space: CGColorSpace(name: CGColorSpace.sRGB)!,
    bitmapInfo: CGBitmapInfo(rawValue: CGImageAlphaInfo.premultipliedLast.rawValue)
)!

context.setFillColor(.red)
context.fill(CGRect(x: 0, y: 0, width: 100, height: 100))

let image = context.makeImage()  // WASMでもネイティブでも動作
```

### Native vs WASM: 根本的な違い

**このライブラリはWASM専用です。** ネイティブプラットフォームではAppleのCoreGraphicsを直接使用します。

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ネイティブ (macOS/iOS/tvOS/watchOS)                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ユーザーコード                                                          │
│       │                                                                 │
│       ▼                                                                 │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │              Apple CoreGraphics (システム提供)                    │   │
│  ├─────────────────────────────────────────────────────────────────┤   │
│  │  • Quartz 2D エンジン                                            │   │
│  │  • ハードウェアアクセラレーション (Metal/GPU)                      │   │
│  │  • フォントレンダリング (Core Text 連携)                          │   │
│  │  • PDF 生成・解析                                                 │   │
│  │  • 画像フォーマット対応 (ImageIO 連携)                            │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  OpenCoreGraphics: 使用しない (canImport(CoreGraphics) = true)         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                              WASM                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ユーザーコード                                                          │
│       │                                                                 │
│       ▼                                                                 │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      OpenCoreGraphics                            │   │
│  ├─────────────────────────────────────────────────────────────────┤   │
│  │  Graphics/ (全アーキテクチャ)                                     │   │
│  │  • CoreGraphics 互換 API (CGContext, CGPath, CGColor, etc.)     │   │
│  │  • 状態管理 (CTM, クリッピング, シャドウ)                          │   │
│  │                                                                  │   │
│  │  Rendering/WebGPU/ (#if arch(wasm32) のみ)                       │   │
│  │  • WebGPU によるGPUレンダリング (自動設定)                        │   │
│  │  • パステッセレーション                                           │   │
│  │  • ブレンドモード (Porter-Duff)                                   │   │
│  │  • グラデーション・シェーディング                                  │   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1amageek/OpenCoreGraphics](https://github.com/1amageek/OpenCoreGraphics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
