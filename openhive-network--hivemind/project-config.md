---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Hivemind** is a HAF (Hive Application Framework)-based microservice that provides social media features for the Hive blockchain. It consists of two main components:

1. **Indexer** (Python): Extracts and processes social data from HAF into application tables
2. **Server** (PL/pgSQL + PostgREST): Exposes a JSON-RPC API for querying social data

The project is transitioning from Python to PL/pgSQL, so you'll find remnants of old code patterns.

## Development Commands

### Environment Setup

**Prerequisites:** Python 3.10+, PostgreSQL 17+, pip >= 22.2.2, setuptools >= 63.1.0

```bash
# Clone with submodules (HAF and reputation_tracker are submodules)
git clone --recurse-submodules https://gitlab.syncad.com/hive/hivemind.git

# Install in virtual environment (RECOMMENDED)
python3 -m venv venv/
. venv/bin/activate
pip install .                    # Base install
pip install .'[dev]'            # With dev tools (ruff, pyYAML)
pip install .'[tests]'          # With testing tools (tox)

# Deactivate virtual environment
deactivate
```

### Building and Running

**Docker Images:**
```bash
# Build HAF (from haf/ submodule)
cd haf
./scripts/ci-helpers/build_instance.sh local $(pwd) registry.gitlab.syncad.com/hive/haf

# Build reputation tracker (from reputation_tracker/ submodule)
cd reputation_tracker
docker build -t registry.gitlab.syncad.com/hive/reputation_tracker:local .

# Build Hivemind
./scripts/ci-helpers/build_instance.sh local $(pwd) registry.gitlab.syncad.com/hive/hivemind

# Build postgrest-rewriter (Nginx)
docker build -t postgrest_rewriter:local -f Dockerfile.rewriter .
```

**Installation on HAF Database:**
```bash
# 1. Setup roles and schema
./scripts/setup_postgres.sh --postgres-url=postgresql://haf_admin@localhost:5432/haf_block_log

# 2. Install reputation_tracker (required dependency)
./reputation_tracker/scripts/install_app.sh \
  --postgres-url=postgresql://haf_admin@localhost:5432/haf_block_log \
  --schema=reptracker_app \
  --is_forking="false"

# 3. Install Hivemind
./scripts/install_app.sh --postgres-url=postgresql://haf_admin@localhost:5432/haf_block_log
```

**Running the Indexer:**
```bash
# Activate virtual environment first
. venv/bin/activate

# Full sync to head block
hive sync \
  --reptracker-schema-name=reptracker_app \
  --database-url=postgresql://hivemind@localhost:5432/haf_block_log

# Sync to specific block (for testing - creates indexes at end)
hive sync \
  --reptracker-schema-name=reptracker_app \
  --test-max-block=5000000 \
  --database-url=postgresql://hivemind@localhost:5432/haf_block_log

# Check available options
hive sync --help
```

**Running the Server:**
```bash
# Start PostgREST server (requires openresty/nginx rewriter)
./scripts/start_postgrest.sh

# Or start openresty separately if installed on host
sudo /etc/init.d/openresty start

# Test the API
curl localhost:8080 \
  --header "Content-Type: application/json" \
  --data '{"id": "test", "method": "condenser_api.get_follow_count", "params": ["gtg"], "jsonrpc": "2.0"}'
```

### Testing

**API Tests (Tavern framework):**
```bash
# Run all API tests
./scripts/run_tests.sh

# Run specific test group (from project root)
./scripts/ci/start-api-smoketest.sh \
  localhost 8080 \
  bridge_api_patterns/get_ranked_posts/ \
  api_smoketest_bridge.xml

# Using tox directly (set environment first)
export HIVEMIND_ADDRESS="localhost"
export HIVEMIND_PORT="8080"
export TAVERN_DIR="$(realpath ./tests/api_tests/hivemind/tavern)"
tox -e tavern -- -W ignore::pytest.PytestDeprecationWarning -n auto --junitxml=results.xml bridge_api_patterns/
```

**Unit Tests:**
```bash
# All tests
make test-all
# or: py.test --cov=hive --capture=sys

# Specific test suites
make test-utils     # Utils tests
make test-server    # Server tests
```

**Test Setup with Mock Data:**

For comprehensive testing, you need HAF synced to 5M blocks + injected mock operations:

```bash
# 1. Start HAF to 5M blocks (requires block_log with 5M+ blocks)
docker run -d -e PG_ACCESS="host haf_block_log all 0.0.0.0/0 trust" \
  --network=haf --name=haf \
  registry.gitlab.syncad.com/hive/haf/minimal-instance:local \
  --replay --stop-at-block=5000000

# 2. Inject mock data (Docker method - includes reputation_tracker + hafah)
docker run --rm --network=haf --name=hivemind \
  registry.gitlab.syncad.com/hive/hivemind/instance:local \
  setup --database-admin-url=postgresql://haf_admin@haf:5432/haf_block_log \
  --with-apps --add-mocks="true"

# 3. Sync reputation_tracker to 4,999,979
docker run --rm --network=haf --name=hivemind \
  --entrypoint=./app/reputation_tracker/scripts/process_blocks.sh \
  registry.gitlab.syncad.com/hive/hivemind/instance:local \
  --stop-at-block=4999979 --postgres-url="postgresql://haf_admin@haf/haf_block_log"

# 4. Sync Hivemind to 5,000,024 (with community-start-block for testing)
docker run --rm --network=haf --name=hivemind \
  registry.gitlab.syncad.com/hive/hivemind/instance:local \
  sync --test-max-block=5000024 --community-start-block=4998000 \
  --database-url=postgresql://hivemind@haf/haf_block_log

# 5. Finish syncing reputation_tracker

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openhive-network/hivemind](https://github.com/openhive-network/hivemind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
