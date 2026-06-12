---
trigger: always_on
description: enableSQL: true,
---

# hallucinate_app Development Guide

## Project Overview

hallucinate_app is an Electron-based desktop application that creates a bridge between IPFS and HuggingFace technologies, enabling decentralized AI model serving, dataset management, and inference. The application leverages a modular architecture with paired JavaScript and Python implementations, secure UCAN-based authentication, efficient Apache Arrow data exchange, and a resource pool pattern for module interdependencies. The system uses multiple database technologies including OrbitDB for peer-to-peer data storage, FireproofDB for serverless database mirroring with CRDT capabilities, and DuckDB for analytical SQL queries with IPLD conversion for peer-to-peer exchange via libp2p.

### Key Technologies
- **Frontend**: Electron (cross-platform desktop application)
- **JavaScript**: ES Modules, Node.can 
- **Python**: Python 3.8+ with HuggingFace ecosystem 
- **AI/ML**: Transformers, PyTorch, FAISS vector database
- **Decentralized**: IPFS, libp2p, OrbitDB, FireproofDB, DuckDB-IPLD
- **Database**: 
  - OrbitDB (P2P event-based)
  - FireproofDB (serverless CRDT)
  - DuckDB (analytical SQL with IPLD conversion for P2P exchange)
- **Build**: Electron Forge
- **Data Exchange**: Apache Arrow C Data Interface, PyBridge
- **Security**: UCAN authentication, capability-based security

## Project Architecture

### Dashboard Integration
The application includes a comprehensive testing dashboard that provides a unified interface for testing and monitoring all modules:

- **Module Testing Panel**: Visual interface for running tests on individual modules
- **Real-time Status Monitoring**: Live status indicators for all core modules
- **Test Orchestration**: Run coordinated tests across multiple modules
- **Database Visualization**: Query and visualize data across all database systems
- **Performance Metrics**: Track performance metrics for all modules
- **Resource Monitoring**: Monitor resource usage across the application
- **Dependency Graph**: Visual representation of module dependencies
- **Test History**: Historical test results with filtering and analysis
- **Security Dashboard**: Manage authentication, capabilities, and API keys
- **Security Status**: Real-time monitoring of authentication and security components
- **Security Testing**: Dedicated tests for auth, keystore, and secured modules

The dashboard provides specialized views for different system aspects:
- **Overview**: System-wide statistics and quick actions
- **Module-specific panels**: Detailed control for individual modules
- **Authentication panel**: Manage principals, capabilities, and API keys
- **Security status section**: Monitor overall system security

The dashboard is accessible both through the Electron application interface and as a standalone web application when running in development mode.

### Core Modules
- **IPFS Kit**: Foundation for IPFS interactions
- **Model Manager**: Manages model loading and serving
- **Transformers**: Integration with HuggingFace transformers
- **Datasets**: Dataset management and processing
- **FAISS**: Vector search capabilities
- **Agents**: AI agent functionality
- **Accelerate**: Performance optimizations
- **libp2p**: P2P communication layer
- **OrbitDB**: Decentralized database storage
- **FireproofDB**: Serverless database mirror/backup with CRDT support
- **DuckDB-IPLD**: Analytical SQL database with IPLD conversion for P2P exchange
- **Authentication**: UCAN-based decentralized authorization
- **Keystore**: Secure storage for API keys and credentials

### Authentication & Security
- ✅ Implementation of UCAN (User Controlled Authorization Networks) for decentralized auth
  - ✅ Custom `ucan_auth_py` package with comprehensive UCAN implementation
  - ✅ Principal, Capability, and Token classes for UCAN operations
  - ✅ Cryptographic functions for secure operations (Ed25519)
  - ✅ DID (Decentralized Identifier) creation and resolution
  - ✅ JavaScript counterpart in `auth.js` with equivalent functionality
- ✅ Three-tier architecture with AuthManager, Keystore, and Integration layers
  - ✅ Python implementation in `auth.py`, `keystore.py`, and `auth_keystore_integration.py`
  - ✅ JavaScript implementation in `auth.js`, `keystore.js`, and `auth_keystore_integration.js`
- ✅ Capability-based security model with principals, permissions, and resources
- ✅ Mock implementation option for development environments
- ✅ Delegated capabilities ensure secure access control across decentralized services
- ✅ Integrated security testing in main testing framework
- ✅ Comprehensive dashboard for managing security components and visualizing status
  - ✅ Security test dashboard UI for running and analyzing security tests
  - ✅ Test history tracking and visualization for security components
  - ✅ Detailed test result display with pass/fail indicators
  - ✅ Integration with notification system for real-time feedback
- ✅ Develop test suites for security validation
  - ✅ Comprehensive test suite for auth, keystore, and secure managers
  - ✅ Visual test runner for security components
  - ✅ Test result history and export capabilities
- ✅ Secure keystore for managing third-party API keys (OpenAI, HuggingFace, etc.)
  - ✅ AES-256-GCM encryption for stored keys in both Python and JavaScript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [endomorphosis/hallucinate_app](https://github.com/endomorphosis/hallucinate_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
