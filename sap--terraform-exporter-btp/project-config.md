---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Terraform Exporter for SAP BTP** (`btptf`) is a CLI tool that exports existing SAP Business Technology Platform (BTP) resources into Terraform code and import blocks. It supports exporting directories, subaccounts, and Cloud Foundry organizations.

The tool operates in two modes:
1. **create-json**: Creates a JSON inventory of resources to export
2. **export-by-json**: Generates Terraform configuration files and import blocks from the JSON inventory

## Build and Development Commands

### Build and Install
```bash
make build          # Build the project
make install        # Build and install btptf to GOPATH/bin
go build -v ./...   # Direct Go build
```

### Testing
```bash
make test                                    # Run all unit tests with coverage
go test -v -cover -timeout=900s -parallel=4 ./...  # Run tests with verbose output
```

### Code Quality
```bash
make lint           # Run golangci-lint
make fmt            # Format code with gofmt -s -w -e .
go fix ./...        # Run go fix (required before commits)
make docs           # Regenerate CLI markdown docs via `go run main.go gendoc` (required after changing any command help text or flags)
```

### Running the CLI
```bash
# Create JSON inventory
go run main.go create-json -s <subaccount-id>

# Export by JSON
go run main.go export-by-json -s <subaccount-id> -p inventory.json

# Use --verbose flag for debugging
go run main.go create-json -s <subaccount-id> --verbose
```

### Debugging in VS Code
Use the "Debug CLI command" launch configuration in `.vscode/launch.json`. Set up environment variables in an `.env` file at the project root with:
- `BTP_USERNAME`
- `BTP_PASSWORD`
- `BTP_GLOBALACCOUNT`

## Architecture

### High-Level Structure

The codebase follows a layered architecture:

1. **cmd/** - Cobra CLI commands and entry points
   - Root command setup and command definitions
   - Commands: `create-json`, `export-by-json`, `export-by-resource`
   - Command help/documentation generation logic

2. **internal/** - Private application code
   - **btpcli/** - SAP BTP API client implementation
     - Client with custom HTTP transport and session management
     - Facade pattern for different BTP services (accounts, security, services)
     - Type definitions for CIS, Service Manager, XSUAA responses
   - **btpclisession/** - Reads an existing `btp` CLI session (config + keychain/file fallback) so the exporter can reuse a CLI login instead of requiring username/password
   - **cfcli/** - Cloud Foundry API client wrapper

3. **pkg/** - Public/reusable packages
   - **tfimportprovider/** - Factory pattern for generating Terraform import blocks per resource type
     - Each resource has its own import provider implementation
     - Factory function `GetImportBlockProvider` routes to the correct implementation
   - **tfutils/** - Terraform operations (init, plan, import, config generation)
   - **tfcleanup/** - Post-processing of generated Terraform code
     - Orchestrator coordinates cleanup across resources
     - Resource processors handle resource-specific transformations
     - Provider processors handle provider block cleanup
   - **output/** - Console output formatting and UI components
   - **files/** - File I/O utilities
   - **resume/** - Export session resumption logic
   - **defaultfilter/** - Default filtering for resource selection
   - **toggles/** - Feature toggle helpers

### Key Architectural Patterns

**Import Provider Factory Pattern**: Each BTP resource type has a dedicated import provider (e.g., `subaccountRoleCollectionImportProvider.go`). The factory function in `tfImportProviderFactory.go` returns the appropriate provider based on resource type.

**BTP Client Facade Pattern**: The `internal/btpcli/client.go` provides low-level HTTP communication with the BTP API. Facade implementations (e.g., `facade_accounts.go`, `facade_security.go`) provide domain-specific methods on top of the base client.

**Cleanup Orchestration**: After generating Terraform configuration, the `tfcleanup/orchestrator` coordinates multiple processors to transform and optimize the generated code. Each resource type can have a dedicated processor for specialized cleanup logic.

## Adding Support for New Resources

To add a new resource type at the subaccount level:

1. **Add constants** in `pkg/tfutils/tfImport.go`:
   - Command parameter name constant (e.g., alongside `CmdSubaccountParameter`)
   - Technical resource name constant

2. **Add mapping** in `TranslateResourceParamToTechnicalName` function in `pkg/tfutils/tfImport.go`

3. **Add to allowed resources** in the matching slice in `pkg/tfutils/tfConfig.go` (`AllowedResourcesSubaccount`, `AllowedResourcesDirectory`, or `AllowedResourcesOrganization`)

4. **Create import provider** in `pkg/tfimportprovider/`:
   - Use `subaccountRoleCollectionImportProvider.go` as a template
   - Implement `CreateImportBlock` method

5. **Register in factory** in `GetImportBlockProvider` function in `pkg/tfimportprovider/tfImportProviderFactory.go`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SAP/terraform-exporter-btp](https://github.com/SAP/terraform-exporter-btp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
