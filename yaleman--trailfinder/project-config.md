---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Trailfinder is a Rust application for network device discovery and configuration parsing. The application can SSH into network devices, automatically identify their brand/type, parse their configurations, and maintain a JSON-based inventory. Features a comprehensive web interface with interactive topology visualization, HTTPS support, and RESTful API. Currently implements MikroTik, Cisco, and Ubiquiti parsers with comprehensive test coverage.

## Commands

### Building

- `cargo build --quiet` - Build the project
- `cargo run --quiet` - Run the main application (device discovery)
- `cargo run --quiet -- web` - Run the web server (HTTP on localhost:8000)
- `cargo run --quiet -- web --tls-cert cert.pem --tls-key key.pem` - Run HTTPS web server
- `cargo run --quiet -- web --help` - Show web server options including TLS configuration

### Testing

- `cargo test --quiet` - Run all tests (253+ tests with comprehensive coverage)
- `cargo test --quiet test_parse_mikrotik` - Run specific test
- `cargo test --quiet mikrotik` - Run all MikroTik parser tests
- `cargo test --quiet cisco` - Run all Cisco parser tests
- `cargo test --quiet lib_tests` - Run core library tests
- `cargo test --quiet main_tests` - Run CLI integration tests
- `cargo test --quiet ssh_tests` - Run SSH client tests

### Development

- `cargo check --quiet` - Fast compile check without producing binaries
- `cargo clippy --all-targets --quiet` - Lint checker
- `cargo fmt` - Format code

### Logging

The application uses structured logging via the `tracing` crate.

Examples:

- `cargo run` - Run with default INFO level logging
- `cargo run -- --debug` - Run with detailed debugging
- `cargo run --release` - Run release build with minimal output

## Architecture

The codebase follows a standard Rust library + binary structure:

- **src/lib.rs** - Core data models and trait definitions:
  - `Device` struct - Main data structure for network devices with interface references
  - `Interface` struct - Network interface with unique ID generation (`interface_id()` method)  
  - `Route` struct - Network routes that reference interfaces by ID
  - `ConfParser` trait - Extensible interface for device configuration parsers
  - Enums for `DeviceType`, `InterfaceType`, `RouteType`, and `Owner`

- **src/main.rs** - Main application logic for device discovery and identification

- **src/config.rs** - JSON-based configuration management:
  - `AppConfig` - Application configuration with device inventory
  - `DeviceConfig` - Per-device configuration including SSH details
  - `DeviceBrand` - Enum for supported device manufacturers

- **src/ssh.rs** - SSH client functionality:
  - `SshClient` - SSH connection with multiple authentication methods
  - `DeviceIdentifier` - Automatic device brand/type detection
  - SSH config integration - reads `~/.ssh/config` for connection details

- **src/brand/mikrotik.rs** - MikroTik-specific parser implementing `ConfParser`
- **src/brand/cisco.rs** - Cisco-specific parser implementing `ConfParser`
- **src/brand/ubiquiti.rs** - Ubiquiti-specific parser implementing `ConfParser`

- **src/web/mod.rs** - Web server and API implementation:
  - `web_server_command` - Main web server function with HTTP/HTTPS support
  - REST API endpoints for devices, topology, and path finding
  - OpenAPI/Swagger documentation generation
  - TLS certificate parsing and hostname extraction
  - HTTPS support with RSA and ECDSA keys (including prime256v1/P-256)

- **src/cli.rs** - Command-line interface with comprehensive subcommands:
  - Device discovery and identification commands
  - Web server startup with TLS configuration options
  - Add/remove device commands
  - Path finding and scanning functionality

## Device Configuration

The application uses a `devices.json` file to store device inventory. Each device entry includes:

- Hostname and IP address  
- SSH connection details (username, port, key path)
- Authentication preferences (ssh-agent, identity files)
- Brand and device type (auto-detected)
- Last interrogation timestamp for caching
- TLS configuration for HTTPS web server (optional):
  - Certificate file path (`tls_cert_file`)
  - Private key file path (`tls_key_file`)
  - Hostname override (`tls_hostname`)

## SSH Authentication

The application supports multiple SSH authentication methods in priority order:

1. **SSH Config Integration** - Automatically reads `~/.ssh/config` for:
   - Username from `User` directive
   - Identity files from `IdentityFile` directive  
   - ssh-agent usage based on `IdentitiesOnly` setting

2. **Manual Configuration** - Fallback to device-specific settings:
   - ssh-agent authentication (default: enabled)
   - SSH key file authentication
   - Password authentication (via `SSH_PASSWORD` environment variable)

The application tries SSH config first, then falls back to manual config if SSH config fails.

## Web Interface

The application includes a comprehensive web interface built with Axum and modern web technologies:

### Core Features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaleman/trailfinder](https://github.com/yaleman/trailfinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
