---
trigger: always_on
description: 1. [Project Overview](#project-overview)
---

# AGENTS.md: Comprehensive Guide for LLM Agentic Development

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture and Core Concepts](#architecture-and-core-concepts)
3. [Repository Structure](#repository-structure)
4. [Setup and Prerequisites](#setup-and-prerequisites)
5. [Core Components](#core-components)
6. [Workflow Guide](#workflow-guide)
7. [Supported Models and Datasets](#supported-models-and-datasets)
8. [Code Examples and Use Cases](#code-examples-and-use-cases)
9. [Advanced Features](#advanced-features)
10. [Best Practices for LLM Agents](#best-practices-for-llm-agents)
11. [Troubleshooting Guide](#troubleshooting-guide)
12. [API Reference](#api-reference)

---

## Project Overview

**MiCo** (Mixed Precision Neural Network Co-Exploration Framework) is an end-to-end framework for training, exploring, and deploying mixed precision quantized models optimized for Edge AI applications.

### Key Capabilities
- **Mixed Precision Quantization (MPQ)**: Automatically search for optimal bitwidth configurations per layer
- **Hardware-Aware Search**: Proxy models for BitFusion and custom hardware latency prediction
- **C Code Generation**: Convert PyTorch models to optimized C code for deployment
- **Multiple Search Algorithms**: Support for Bayesian Optimization, HAQ, NLP, and custom MiCo searcher
- **End-to-End Pipeline**: From training to hardware deployment

### Publication
The framework is described in the paper: "MiCo: End-to-End Mixed Precision Neural Network Co-Exploration Framework for Edge AI" (ICCAD 2025)

---

## Architecture and Core Concepts

### Mixed Precision Quantization (MPQ)
- **Per-Layer Bitwidth Assignment**: Each layer can have different weight and activation bitwidths (e.g., 1-8 bits)
- **Search Space**: Explores combinations of bitwidths to find optimal accuracy-efficiency tradeoffs
- **Constraints**: Can optimize under BOPs (Bit Operations), MACs, or hardware latency constraints

### Three-Stage Pipeline
1. **Training Stage**: Train/load a full-precision or quantized model
2. **Search Stage**: Use MPQ search algorithms to find optimal bitwidth configurations
3. **Deployment Stage**: Generate C code and compile for target hardware

### Hardware-Aware Features
- **Proxy Models**: Predict hardware latency without running on actual hardware
- **Supported Targets**: CPUs (RISC-V VexiiRiscv, Rocket, Boom), Gemmini, BitFusion
- **Chipyard Integration**: Compatible with Chipyard ecosystem designs

---

## Repository Structure

```
MiCo-python/
├── Core Modules (Python files in root)
│   ├── MiCoModel.py          # Base model class with unified training/testing
│   ├── MiCoQLayers.py         # Quantized layer implementations (BitLinear, BitConv2d, etc.)
│   ├── MiCoUtils.py           # Utilities for layer replacement, model fusion, export
│   ├── MiCoEval.py            # Model evaluation (accuracy, BOPs, MACs, latency)
│   ├── MiCoSearch.py          # MPQ search coordination
│   ├── MiCoCodeGen.py         # C code generator using Torch FX
│   ├── MiCoGraphGen.py        # DNN Weaver graph generator
│   ├── MiCoLLaMaGen.py        # Specialized LLaMa C code generator
│   ├── MiCoProxy.py           # Hardware latency proxy models
│   ├── MiCoRegistry.py        # Registry pattern for custom operations
│   ├── MiCoAnalysis.py        # Model statistics and analysis
│   ├── SimUtils.py            # Hardware simulation utilities
│   ├── DimTransform.py        # Dimension transformation for search
│   └── MiCoDatasets.py         # Dataset loaders
│
├── searchers/                 # MPQ search algorithms
│   ├── MiCoSearcher.py        # Main MiCo searcher (RF/XGB/Bayes)
│   ├── BayesSearcher.py       # Bayesian Optimization
│   ├── HAQSearcher.py         # HAQ (Hardware-Aware Quantization)
│   ├── NLPSearcher.py         # Natural Language Processing inspired
│   ├── RegressionSearcher.py  # Regression-based searcher
│   ├── QSearcher.py           # Base searcher class
│   └── SearchUtils.py         # Sampling utilities
│
├── models/                    # Model architectures
│   ├── MLP.py, LeNet.py, VGG.py, ResNet.py, MobileNetV2.py
│   ├── SqueezeNet.py, ShuffleNet.py, LLaMa.py, ViT.py
│   ├── DSCNN.py, M5.py, HARMLP.py
│   └── model_zoo.py           # Model registry
│
├── examples/                  # Example training and search scripts
│   ├── lenet_mnist.py         # Train LeNet on MNIST
│   ├── lenet_mnist_search.py  # MPQ search on LeNet
│   ├── mpq_search.py          # General MPQ search script
│   ├── mpq_train.py           # General MPQ training script
│   └── [other model examples]
│
├── deploy/                    # Hardware deployment examples
│   ├── lenet_on_mico.py       # Deploy LeNet on MiCo hardware
│   ├── vgg_on_bf.py           # Deploy VGG on BitFusion
│   └── [other deployment scripts]
│
├── doc/                       # Documentation
│   ├── REGISTRY_USAGE.md      # Guide for custom operations
│   ├── MEMORY_OPTIMIZATION.md # Memory pool optimization details
│   ├── CHIPYARD_INTEGRATION.md # Chipyard integration guide
│   └── icon_v1.jpg
│
├── tests/                     # Unit and integration tests
├── project/                   # C project templates (requires MiCo Library submodule)
├── hw/                        # Hardware-specific implementations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HKUSTGZ-MICS-LYU/MiCo-python](https://github.com/HKUSTGZ-MICS-LYU/MiCo-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
