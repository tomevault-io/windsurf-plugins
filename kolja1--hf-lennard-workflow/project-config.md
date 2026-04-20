---
trigger: always_on
description: This is the Rust-based workflow system for the HF-Lennard project, which processes LinkedIn contacts through an automated business letter generation pipeline. The system integrates with multiple services including Zoho CRM, a dossier extraction service, a letter generation AI service, and PDF generation.
---

# CLAUDE.md - AI Assistant Documentation

## System Overview

This is the Rust-based workflow system for the HF-Lennard project, which processes LinkedIn contacts through an automated business letter generation pipeline. The system integrates with multiple services including Zoho CRM, a dossier extraction service, a letter generation AI service, and PDF generation.

## Architecture

### Core Components

1. **Workflow Orchestrator** (`crates/workflow-core/src/workflow/orchestrator.rs`)
   - Manages the complete workflow pipeline
   - Handles error recovery and notifications
   - Uses trait-based steps for testability

2. **gRPC Services**
   - **Workflow Service** (port 50051): Receives triggers from Telegram bot
   - **Dossier Service** (port 50052): Extracts company/person information  
   - **Letter Service** (port 50053): Generates personalized letters using AI
   - **PDF Service** (port 8000): Converts ODT templates to PDF

3. **Key Clients** (`crates/workflow-core/src/clients/`)
   - `zoho.rs`: Zoho CRM integration via Nango
   - `dossier.rs`: Company dossier extraction
   - `letter_service.rs`: AI letter generation
   - `pdf.rs`: PDF generation from templates
   - `telegram.rs`: Approval requests and notifications

## Development Guidelines

### ✅ DOs - Best Practices from User Feedback

1. **ALWAYS run linting/typecheck after changes**
   ```bash
   cargo clippy
   cargo check
   ```
   If these commands aren't found, ask the user for the correct commands and save them here.

2. **Use gRPC services for complex operations**
   - Letter generation → gRPC Letter Service
   - Dossier extraction → gRPC Dossier Service
   - Don't implement complex logic directly in workflow

3. **Make paths configurable**
   - Use command-line arguments for paths (data-dir, logs-dir, templates-dir)
   - Support both Docker and local development
   - Use `OnceCell` for one-time initialization

4. **Log everything important**
   - Log gRPC requests/responses to files for debugging
   - Use structured logging with context
   - Log to mounted volumes for persistence

5. **Handle errors gracefully**
   - Send Telegram notifications on errors
   - Update task status in Zoho
   - Provide detailed error context

6. **Test locally before Docker**
   - Use `scripts/run-local.sh` for rapid iteration
   - Test with actual services running
   - Verify data flow through entire pipeline

### ❌ DON'Ts - Avoid These Patterns

1. **DON'T use environment variables for configuration**
   - Use JSON config files instead
   - Pass paths via command-line arguments

2. **DON'T hardcode paths**
   - All paths should be configurable
   - Use the `paths` module for consistency

3. **DON'T suppress warnings**
   - Fix all compiler warnings properly
   - Address the root cause, not symptoms

4. **DON'T create files unless necessary**
   - Prefer editing existing files
   - Never create documentation files proactively

5. **DON'T use placeholder/template text**
   - Always generate real, personalized content
   - Use the appropriate gRPC service

6. **DON'T commit without user request**
   - Only commit when explicitly asked
   - Always run tests before committing

## Deployment & Testing

### Local Development Setup

1. **Start required services:**
   ```bash
   # Start PDF generator
   cd ../hf-lennard/libreoffice-pdf-generator
   docker compose up -d
   
   # Start dossier service  
   cd ../hf-lennard-dossier-extraction-production
   docker compose up -d
   
   # Start letter generation service
   cd ../hf-lennard-letter-writer-ai
   docker compose up -d
   ```

2. **Run workflow locally:**
   ```bash
   # Build and run with local data directories
   ./scripts/run-local.sh
   ```

3. **Directory structure:**
   ```
   docker/volumes/
   ├── workflow-data/     # Workflow data files
   │   ├── triggers/      # Incoming triggers
   │   ├── processed/     # Completed workflows
   │   └── data/          # Dossiers, letters, etc.
   └── logs/              # gRPC logs for debugging
       └── grpc/
           └── dossier/   # Request/response logs
   ```

### Docker Deployment

1. **Build and run:**
   ```bash
   docker compose up --build
   ```

2. **Configuration:**
   - Mount `./config/credentials.json` for API keys
   - Mount `./templates/` for ODT templates
   - Data persists in `docker/volumes/`

3. **Monitoring:**
   ```bash
   # Check logs
   docker logs rust-workflow-server -f
   
   # Check gRPC logs
   ls -la docker/volumes/logs/grpc/dossier/
   ```

### Testing Workflow

1. **Trigger via Telegram:**
   - Bot receives task from user
   - Sends gRPC trigger to workflow (port 50051)
   - Monitor progress in Telegram chat

2. **Check service health:**
   ```bash
   # PDF service
   curl http://localhost:8000/health
   
   # Check if services are running
   docker ps | grep -E "dossier|letter|pdf"
   
   # Check ports
   netstat -tuln | grep -E "50051|50052|50053|8000"
   ```

3. **Debug issues:**
   - Check gRPC request/response logs in `docker/volumes/logs/`
   - Review Telegram error messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kolja1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
