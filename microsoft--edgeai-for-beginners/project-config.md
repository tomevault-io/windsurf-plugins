---
trigger: always_on
description: > **Developer Guide for Contributing to EdgeAI for Beginners**
---

# AGENTS.md

> **Developer Guide for Contributing to EdgeAI for Beginners**
> 
> This document provides comprehensive information for developers, AI agents, and contributors working with this repository. It covers setup, development workflows, testing, and best practices.
> 
> **Last Updated**: October 30, 2025 | **Document Version**: 3.0

## Table of Contents

- [Project Overview](#project-overview)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Setup Commands](#setup-commands)
- [Development Workflow](#development-workflow)
- [Testing Instructions](#testing-instructions)
- [Code Style Guidelines](#code-style-guidelines)
- [Pull Request Guidelines](#pull-request-guidelines)
- [Translation System](#translation-system)
- [Foundry Local Integration](#foundry-local-integration)
- [Build and Deployment](#build-and-deployment)
- [Common Issues and Troubleshooting](#common-issues-and-troubleshooting)
- [Additional Resources](#additional-resources)
- [Project-Specific Notes](#project-specific-notes)
- [Getting Help](#getting-help)

## Project Overview

EdgeAI for Beginners is a comprehensive educational repository teaching Edge AI development with Small Language Models (SLMs). The course covers EdgeAI fundamentals, model deployment, optimization techniques, and production-ready implementations using Microsoft Foundry Local and various AI frameworks.

**Key Technologies:**
- Python 3.8+ (primary language for AI/ML samples)
- .NET C# (AI/ML Samples)
- JavaScript/Node.js with Electron (for desktop applications)
- Microsoft Foundry Local SDK
- Microsoft Windows ML 
- VSCode AI Toolkit
- OpenAI SDK
- AI Frameworks: LangChain, Semantic Kernel, Chainlit
- Model Optimization: Llama.cpp, Microsoft Olive, OpenVINO, Apple MLX

**Repository Type:** Educational content repository with 8 modules and 10 comprehensive sample applications

**Architecture:** Multi-module learning path with practical samples demonstrating edge AI deployment patterns

## Repository Structure

```
edgeai-for-beginners/
├── introduction.md          # Course introduction and overview
├── Module01-07/            # Core educational modules (Markdown)
├── Module08/               # Foundry Local toolkit with 10 samples
│   ├── samples/01-06/     # Foundation samples (Python)
│   ├── samples/07/        # API client (Python)
│   ├── samples/08/        # Windows 11 chat app (Electron)
│   └── samples/09-10/     # Advanced multi-agent systems (Python)
├── Workshop/               # Hands-on workshop materials
│   ├── samples/           # Workshop Python samples with utilities
│   │   ├── session01/     # Chat bootstrap samples
│   │   ├── session02-06/  # Progressive workshop sessions
│   │   └── util/          # Workshop utility modules
│   ├── notebooks/         # Jupyter notebook tutorials
│   └── scripts/           # Validation and testing tools
├── translations/          # Multi-language translations (50+ languages)
├── translated_images/     # Localized images
└── imgs/                  # Course images and assets
```

## Prerequisites

### Required Tools

- **Python 3.8+** - For AI/ML samples and notebooks
- **Node.js 16+** - For Electron sample application
- **Git** - For version control
- **Microsoft Foundry Local** - For running AI models locally

### Recommended Tools

- **Visual Studio Code** - With Python, Jupyter, and Pylance extensions
- **Windows Terminal** - For better command-line experience (Windows users)
- **Docker** - For containerized development (optional)

### System Requirements

- **RAM**: 8GB minimum, 16GB+ recommended for multi-model scenarios
- **Storage**: 10GB+ free space for models and dependencies
- **OS**: Windows 10/11, macOS 11+, or Linux (Ubuntu 20.04+)
- **Hardware**: CPU with AVX2 support; GPU (CUDA, Qualcomm NPU) optional but recommended

### Knowledge Prerequisites

- Basic understanding of Python programming
- Familiarity with command-line interfaces
- Understanding of AI/ML concepts (for sample development)
- Git workflows and pull request processes

## Setup Commands

### Repository Setup

```bash
# Clone the repository
git clone https://github.com/microsoft/edgeai-for-beginners.git
cd edgeai-for-beginners

# No build step required - this is primarily an educational content repository
```

### Python Sample Setup (Module08 and Workshop samples)

```bash
# Create and activate virtual environment
python -m venv .venv
# On Windows
.venv\Scripts\activate
# On macOS/Linux
source .venv/bin/activate

# Install Foundry Local SDK and dependencies
pip install foundry-local-sdk openai

# Install additional dependencies for Module08 samples
cd Module08
pip install -r requirements.txt

# Install Workshop dependencies
cd ../Workshop
pip install -r requirements.txt
```

### Node.js Sample Setup (Sample 08 - Windows Chat App)

```bash
cd Module08/samples/08
npm install

# Start in development mode
npm run dev

# Build for production
npm run build

# Create installer
npm run dist
```

### Foundry Local Setup

Foundry Local is required to run the samples. Download and install from the official repository:

**Installation:**
- **Windows**: `winget install Microsoft.FoundryLocal`
- **macOS**: `brew tap microsoft/foundrylocal && brew install foundrylocal`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/edgeai-for-beginners](https://github.com/microsoft/edgeai-for-beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
