---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **kafka-docker-playground**, a comprehensive testing framework for Apache Kafka and Confluent Platform. It provides:
- 170+ self-managed connector examples (`connect/` directory)
- 100+ Confluent Cloud fully-managed connector examples (`ccloud/` directory)
- Reproduction models for testing and debugging (`reproduction-models/` - private submodule)
- Multiple secured environments (SASL, RBAC, SSL, Kerberos, etc. in `environment/`)
- A powerful CLI tool (`scripts/cli/playground`)

Documentation: https://kafka-docker-playground.io/

## Common Commands

### Running Examples

```bash
# Run an example interactively (recommended)
playground run

# Run a specific example
cd connect/connect-aws-s3-sink
./s3-sink.sh

# Re-run the last example
playground re-run

# Stop currently running example
playground stop

# View run history and rerun
playground history
```

### Environment Management

```bash
# Start a specific environment (called from within example scripts)
playground start-environment --environment plaintext
playground start-environment --environment sasl-plain
playground start-environment --environment 2way-ssl

# Specify Confluent Platform version
playground run --tag 7.5.0

# Update versions of running components
playground update-version --tag 7.6.0
```

### Connector Operations

```bash
# Create or update a connector (used in example scripts)
playground connector create-or-update --connector <name> << EOF
{
  "connector.class": "...",
  ...
}
EOF

# List running connectors
playground connector status

# Show connector config
playground connector show-config --connector <name>

# Delete a connector
playground connector delete --connector <name>
```

### Schema Registry

```bash
# Get all schema versions for a subject
playground schema get --subject <subject-name>

# Register a schema
playground schema register --subject <subject> --schema-file <file>

# Get/set compatibility level
playground schema get-compatibility --subject <subject>
playground schema set-compatibility --subject <subject> --compatibility BACKWARD
```

### Debugging

```bash
# Enable remote debugging on a container
playground debug enable-remote-debugging --container connect

# Take thread dump
playground debug thread-dump --container connect

# Take heap dump
playground debug heap-dump --container connect

# Analyze heap dump
playground debug heap-analyze --heap-dump-file-path <path>

# TCP dump (network sniffing)
playground debug tcp-dump --container connect

# Generate diagnostics bundle
playground debug generate-diagnostics
```

### Container Operations

```bash
# Get container logs
playground container logs --container <name>

# Execute command in container
playground container exec --container <name> --command "<command>"

# Get JMX metrics
playground get-jmx-metrics --container <name>
```

## Architecture

### Example Structure

Each connector example follows this pattern:

1. **Source `scripts/utils.sh`**: Loads core utility functions and sets default environment variables (TAG, CP versions, etc.)

2. **Start Environment**: Uses `playground start-environment --environment <type>` with optional docker-compose overrides

3. **Configure Resources**: Creates topics, schemas, external resources (S3 buckets, databases, etc.)

4. **Create Connector**: Uses `playground connector create-or-update` to deploy the connector

5. **Verify**: Produces/consumes data to verify functionality

Example script structure:
```bash
#!/bin/bash
set -e

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null && pwd )"
source ${DIR}/../../scripts/utils.sh

# Start environment
playground start-environment --environment plaintext

# Setup resources
# ... create topics, external resources, etc ...

# Create connector
playground connector create-or-update --connector my-connector << EOF
{
  "connector.class": "io.confluent.connect.SomeConnector",
  ...
}
EOF
```

### Directory Organization

- **`connect/connect-<name>/`**: Self-managed connector examples
  - Multiple `.sh` files for different scenarios
  - `docker-compose.plaintext*.yml` for environment-specific overrides
  - Optional subdirectories for custom code/configs
  
- **`ccloud/fm-<name>/`**: Confluent Cloud fully-managed connector examples

- **`reproduction-models/`**: Git submodule with private reproduction models
  - Organized by connector/feature
  - Each contains standalone reproduction scenarios

- **`environment/<type>/`**: Base Kafka environments
  - `plaintext/`: No authentication
  - `sasl-plain/`, `sasl-ssl/`, `2way-ssl/`, `kerberos/`: Secured environments
  - `rbac-sasl-plain/`: RBAC enabled
  - `mdc-*/`: Multi-datacenter configurations

- **`scripts/utils.sh`**: Core utility functions
  - Version handling (CP versions, connector versions)
  - Logging functions (`log`, `logwarn`, `logerror`)
  - AWS/Azure/GCP credential handling
  - Connector installation utilities

- **`scripts/cli/`**: Playground CLI implementation
  - `src/commands/`: Individual CLI commands
  - `src/lib/`: Shared libraries

### Environment Variables

Key environment variables (set in `scripts/utils.sh`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vdesabou/kafka-docker-playground](https://github.com/vdesabou/kafka-docker-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
