---
trigger: always_on
description: This document provides guidance for AI agents (like GitHub Copilot, Claude, or other AI assistants) working on the MAVLink Link Tester project.
---

# AI Agent Guide for MAVLink Link Tester

This document provides guidance for AI agents (like GitHub Copilot, Claude, or other AI assistants) working on the MAVLink Link Tester project.

## Project Overview

**MAVLink Link Tester** is a Python tool for characterizing MAVLink connection reliability and latency. It monitors multiple MAVLink links concurrently, tracking packet loss, latency (RTT), out-of-order packets, and link outages.

### Key Capabilities
- Simultaneous testing of multiple MAVLink connections (UDP, TCP, Serial)
- Real-time latency measurement using TIMESYNC messages
- Packet loss and sequence tracking
- Link outage detection with configurable timeouts
- MAVLink 2.0 signing support
- CSV output with per-second metrics
- Histogram generation for statistical analysis

## Architecture

### Core Components

```
mavlinklinktester/
├── mavlink_link_tester.py    # Main entry point, CLI handling, multi-link coordination
├── link_monitor.py            # Single link monitoring (latency, packets, outages)
├── histogram_generator.py     # Statistical histogram generation
├── connection/
│   ├── mavconnection.py       # Base MAVLink connection class (asyncio Protocol)
│   ├── udplink.py            # UDP transport implementation
│   ├── tcplink.py            # TCP transport implementation
│   └── seriallink.py         # Serial transport implementation
└── mavlink/
    └── pymavutil.py          # MAVLink utilities (signing, message handling)
```

### Key Design Patterns

1. **Asyncio-based**: All I/O operations use Python's asyncio for concurrent operation
2. **Protocol Pattern**: Connection classes implement asyncio's Protocol interface
3. **Callback Pattern**: Message reception uses callbacks for non-blocking processing
4. **Per-second metrics**: Statistics are accumulated and reported every second

### Data Flow

```
MAVLinkLinkTester (main)
    └─> LinkMonitor (per connection)
        └─> MAVConnection (UDP/TCP/Serial)
            └─> asyncio Protocol callbacks
                └─> _on_message_received()
                    ├─> Sequence tracking
                    ├─> Latency measurement
                    └─> Outage detection
```

## Development Workflow

### Setup Development Environment

```bash
# Install with development dependencies
poetry install --with dev

# Activate virtual environment (optional)
poetry shell
```

### Running Tests

```bash
# Run all tests
poetry run pytest

# Run specific test file
poetry run pytest tests/test_link_monitor.py

# Run with coverage
poetry run pytest --cov=mavlinklinktester --cov-report=html

# Run single test
poetry run pytest tests/test_link_monitor.py::TestLinkMonitor::test_outage_detection_entry -v
```

### Code Quality Checks

```bash
# Linting with flake8
poetry run flake8 mavlinklinktester/ tests/

# Type checking with mypy
poetry run mypy mavlinklinktester/

# Dead code detection with vulture
poetry run vulture mavlinklinktester/ --min-confidence 80
```

### Running the Tool

```bash
# Basic usage
poetry run mavlink-link-tester.py --system-id 1 --component-id 1 udpin:0.0.0.0:14550

# Multiple links
poetry run mavlink-link-tester.py --system-id 1 --component-id 1 \
    udpin:0.0.0.0:14550 \
    udpout:192.168.1.100:14551
```

## Key Implementation Details

### Latency Measurement

- Uses MAVLink TIMESYNC messages sent at 2Hz
- Calculates round-trip time (RTT) in milliseconds
- Stores value of -1.0 when no measurement available in current second
- **Important**: -1 values are filtered out when calculating statistics

### Sequence Tracking

- MAVLink messages have 8-bit sequence numbers (0-255, wraps around)
- Out-of-order packets are detected and tracked separately from drops
- Pending sequences more than 50 packets old are considered truly dropped
- Last sequence is tracked per connection
- Total packet count is maintained to track age of pending sequences

### Outage Detection

- Outage triggered when no packets received for `outage_timeout` seconds (default: 1.0s)
- Recovery requires `recovery_hysteresis` consecutive packets (default: 3)
- **Critical**: Outage duration is counted even if program stops during outage
- Total outage time is accumulated across multiple outage events

### MAVLink Signing

- Supports MAVLink 2.0 message signing for authenticated connections
- Uses SHA-256 hashing of passphrase
- Configurable per-link with `--signing-passphrase` and `--signing-link-id`

## Testing Guidelines

### Test Structure

Tests use pytest with asyncio support:
- Fixtures for common setup (monitor instances, temp directories)
- Mock objects for protocol components (connections, tasks)
- Async tests marked with `@pytest.mark.asyncio`

### Important Test Patterns

1. **Mocking connections**: Use `Mock()` for connection objects to avoid real I/O
2. **Time simulation**: Use `time.time()` offsets for testing time-based behavior
3. **Message mocking**: Create mock messages with `get_seq()`, `get_type()`, etc.
4. **Testing async methods**: Always use `await` and `@pytest.mark.asyncio`

### Example Test Pattern

```python
@pytest.mark.asyncio
async def test_something(self, monitor):
    # Mock dependencies
    monitor.connection = Mock()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stephendade/mavlinklinktester](https://github.com/stephendade/mavlinklinktester) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
