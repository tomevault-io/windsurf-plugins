---
trigger: always_on
description: - [Project Overview](#project-overview)
---

# CSV2Notion Neo - Agent Documentation

## Table of Contents

- [Project Overview](#project-overview)
- [Version 2.0.0+ Migration](#version-200-migration)
  - [Migration Details](#migration-details)
    - [Authentication Changes](#authentication-changes)
    - [API Implementation Changes](#api-implementation-changes)
    - [Database URL Requirements](#database-url-requirements)
    - [File Upload Implementation](#file-upload-implementation)
    - [Error Handling Improvements](#error-handling-improvements)
    - [Codebase Structure Changes](#codebase-structure-changes)
- [Core Architecture](#core-architecture)
  - [Main Components](#main-components)
- [Key Features](#key-features)
  - [Data Import Capabilities](#data-import-capabilities)
  - [Advanced Operations](#advanced-operations)
  - [AI Integration](#ai-integration)
  - [Performance Features](#performance-features)
- [Development Guidelines](#development-guidelines)
  - [Code Structure](#code-structure)
  - [Local Development Workflow](#local-development-workflow)
  - [Testing Strategy](#testing-strategy)
  - [Code Quality](#code-quality)
  - [Dependencies](#dependencies)
- [API Integration](#api-integration)
  - [Notion API](#notion-api)
  - [Hugging Face API](#hugging-face-api)
- [Configuration Management](#configuration-management)
  - [Command Line Arguments](#command-line-arguments)
  - [Environment Variables](#environment-variables)
- [Error Handling](#error-handling)
  - [Custom Exceptions](#custom-exceptions)
  - [Logging Strategy](#logging-strategy)
- [Security Considerations](#security-considerations)
  - [Authentication](#authentication)
  - [Data Privacy](#data-privacy)
- [Performance Optimization](#performance-optimization)
  - [Upload Efficiency](#upload-efficiency)
  - [Memory Management](#memory-management)
- [Deployment and Distribution](#deployment-and-distribution)
  - [Packaging](#packaging)
  - [CI/CD Pipeline](#cicd-pipeline)
- [Maintenance and Updates](#maintenance-and-updates)
  - [Version Management](#version-management)
  - [API Compatibility](#api-compatibility)
- [Contributing Guidelines](#contributing-guidelines)
  - [Development Setup](#development-setup)
  - [Ephemeral Build System](#ephemeral-build-system)
  - [Code Review Process](#code-review-process)
  - [Testing Requirements](#testing-requirements)
- [Troubleshooting](#troubleshooting)
  - [Common Issues](#common-issues)
  - [Debug Mode](#debug-mode)
- [Future Roadmap](#future-roadmap)
  - [Planned Features](#planned-features)
  - [Technical Debt](#technical-debt)
- [Dev Toolchain](#dev-toolchain)

## Project Overview

CSV2Notion Neo is an advanced command-line tool for uploading and merging CSV or JSON files with images to Notion databases. It serves as a successor to the original csv2notion project, providing enhanced compatibility with Notion's evolving API and additional features like AI-powered image captioning.

## Version 2.0.0+ Migration

CSV2Notion Neo has been fully migrated to use the official [Notion API](https://developers.notion.com/) (version 2025-09-03) and the [notion-sdk-py](https://github.com/ramnes/notion-sdk-py) library (`notion-client` ^3.1.0). This migration provides better reliability, security, and future compatibility. The application now uses Notion integration tokens instead of session cookies and requires database URLs to be provided for all operations.

### Migration Details

#### Authentication Changes
- Previous: Used `token_v2` session cookies from Notion web interface
- Current: Uses official Notion integration tokens with format validation (starts with 'secret_' or 'ntn_')
- Benefits: Enhanced security, better token management, official authentication method, improved user experience with clear error messages

#### API Implementation Changes
- Previous: Custom implementation using unofficial private Notion APIs
- Current: Official [notion-sdk-py](https://github.com/ramnes/notion-sdk-py) library integration (`notion-client` ^3.1.0)
- API Version: Notion API 2025-09-03 with data_sources structure
- Data Sources: Database properties retrieved from data_sources endpoint instead of database object
- Schema Updates: Database schema modifications use data_sources.update endpoint
- Database Creation: Uses initial_data_source for new database property definitions
- Benefits: Official API support, better error handling, future-proof compatibility

#### Database URL Requirements
- Previous: Optional database URL with automatic database creation
- Current: Mandatory database URL for all operations (supports both database URLs and page URLs)
- Benefits: Clearer workflow, explicit database targeting, better error handling, ability to create databases within existing pages

#### File Upload Implementation
- Previous: Custom file upload logic with signed URLs
- Current: Official Notion file_uploads API endpoints
- Benefits: Standardised upload process, better error handling, official support

#### Error Handling Improvements
- Previous: Custom error handling for unofficial API responses
- Current: Official API error codes and structured error responses
- Benefits: Better error messages, standardised error handling, official documentation

#### Codebase Structure Changes
- Removed: `csv2notion_neo/notion/` directory (legacy API implementation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAcharya/csv2notion-neo](https://github.com/TheAcharya/csv2notion-neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
