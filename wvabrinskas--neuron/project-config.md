---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Maintaining this document:** After making large changes to the codebase, update this file with references to those changes. Keep the architecture, debugging patterns, and common pitfalls sections current so future agents have accurate context.

**Document placement:** When creating agent references, migration summaries, or other markdown documentation, place them in the `docs/` directory rather than the project root.

## Recent Changes (Reference for Updates)

- **Pointer-based arithmetic migration**: All layers and optimizers now use `TensorStorage` / `TensorStorage.Pointer` (`UnsafeMutablePointer<Tensor.Scalar>`) and `NumSwiftFlat` pointer APIs instead of `Tensor.Value` (ContiguousArray) arithmetic. See "Pointer-Based Arithmetic" section below.
- **Tensor batching**: `TensorSize` now has a `batchCount` field. `[Tensor].asTensor` packs an array into a single batched tensor. `tensor.batchSlice(b)` extracts a single batch. Axis 3 in `tensor.adding(tensor:axis:)` concatenates along the batch dimension.
- **Optimizer state uses TensorStorage**: Adam, SGD, and RMSProp store momentum/velocity as `[TensorStorage]` instead of `[Tensor.Value]`. SGD returns `forceCopy()` to avoid shared-memory bugs.
- **Device protocol expanded**: `Device` now has pointer-based `conv2d` and `transConv2d` methods accepting `TensorStorage.Pointer` directly.
- **InstanceNormalize fix**: Gradient layout was reversed (gamma|beta vs beta|gamma). Backward now returns `dBeta.concat(dGamma)` to match `weights`; `apply()` uses `depthSlice(0)` for beta, `depthSlice(1)` for gamma. See `InstanceNormalize.swift`.
- **Tensor.flatArray → asArray**: Renamed for consistency with `TensorSize.asArray`. Use `tensor.asArray` for flat `Tensor.Value`.

## Overview

Neuron is a Swift-based machine learning framework built from scratch for iOS, macOS, tvOS, and watchOS. It implements neural networks with custom backpropagation, supporting various architectures including CNNs, RNNs, LSTMs, GANs, and more. The framework runs on CPU with C-level optimizations via NumSwift.

## Build & Test Commands

### Building
```bash
swift build
```

### Testing
```bash
# Run all tests (always use CI=true)
CI=true swift test

# Run specific test
CI=true swift test --filter <TestName>
```

### Performance Note
Neuron runs ~10X faster in RELEASE mode due to compiler optimizations. For development:
```bash
swift build -c release
```

### Onboarding
Before development, install Xcode templates:
```bash
./scripts/onboard.sh
```

## Architecture

### Core Components

#### Tensor (Sources/Neuron/Tensor/)
- **Tensor**: The fundamental tensor type backed by flat `ContiguousArray<Scalar>` storage with `TensorSize` metadata
- **TensorStorage**: Reference-counted wrapper around `UnsafeMutablePointer<Tensor.Scalar>`. Use `TensorStorage.create(count:)` to allocate and `storage.pointer` for raw access. Prefer constructing tensors with `Tensor(storage:size:)` over `Tensor(Tensor.Value, size:)`.
- **TensorStorage.Pointer**: Typealias for `UnsafeMutablePointer<Tensor.Scalar>`. Use pointer arithmetic (`ptr + offset`) to navigate depth slices and batch slices.
- **Tensor.depthPointer(_:)**: Returns a `TensorStorage.Pointer` to the start of a depth slice without copying. Prefer over `depthSlice(_:)` (which copies into a `Tensor.Value`) in hot paths.
- **Tensor.asArray**: Bridge property returning flat `Tensor.Value` (ContiguousArray); prefer `storage` in hot paths. Formerly `flatArray`.
- **Tensor.value**: Legacy nested `[[[Scalar]]]` view (reconstructed on access); prefer `storage` + `size` in hot paths
- **TensorContext**: Holds backpropagation function for gradient computation
- **TensorSize**: Defines tensor dimensions as `(columns, rows, depth, batchCount)`. `batchCount` defaults to 1. `unitSize` returns `[columns, rows, depth]` without batch.
- Supports automatic gradient calculation via `.gradients(delta:wrt:)` method
- Arithmetic operators overloaded for element-wise and tensor operations
- **Batching**: `[Tensor].asTensor` packs tensors into a single batched tensor. `tensor.batchSlice(b)` extracts one batch. Axis 3 concatenates along batch dimension.

#### Layers (Sources/Neuron/Layers/)
All layers inherit from `BaseLayer` and conform to the `Layer` protocol:
- **BaseLayer**: Base class handling batch processing, device management, weight initialization
- **EncodingType**: Enum defining all layer types for serialization
- Layer categories:
  - Convolutional: `Conv2d`, `TransConv2d`, `MaxPool`, `AvgPool`
  - Dense: `Dense`, `Flatten`, `Reshape`
  - Normalization: `BatchNormalize`, `LayerNormalize`, `InstanceNormalize`
  - Activation: `ReLu`, `LeakyReLu`, `Sigmoid`, `Softmax`, `Tanh`, `Swish`, `SeLu`, `GeLu`
  - Regularization: `Dropout`
  - Recurrent: `LSTM`, `LSTMCell`
  - Other: `Embedding`

#### Trainable (Sources/Neuron/Trainable/)
- **Sequential**: Main network container that chains layers and manages forward/backward passes
- Implements result builder pattern: `Sequential { [Layer1(), Layer2(), ...] }`
- Handles automatic input size propagation through layers
- Supports model import/export via `.smodel` files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wvabrinskas/Neuron](https://github.com/wvabrinskas/Neuron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
