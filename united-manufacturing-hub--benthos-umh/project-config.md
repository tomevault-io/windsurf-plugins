---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the benthos-umh repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the benthos-umh repository.

## Repository Purpose

benthos-umh is a specialized extension of Benthos (Redpanda Connect) tailored for the manufacturing industry. It serves as the stream processing engine for the United Manufacturing Hub, bridging industrial protocols with the UNS (Unified Namespace) through YAML-defined data pipelines.

## Documentation Structure

- `docs/input/` - Input connectors documentation
  - OPC UA, Modbus, S7, Sparkplug B, Ethernet/IP, sensorconnect
- `docs/output/` - Output connectors documentation
  - UNS, OPC UA, Sparkplug B
- `docs/processing/` - Processing pipelines documentation
  - Tag processor, downsampler, stream processor, topic browser
- `docs/libraries/` - Shared libraries documentation
  - UMH topic parser
- `docs/testing/` - Testing approach and examples
- Full documentation: [docs.umh.app/benthos-umh](https://docs.umh.app/benthos-umh)

## Plugin Architecture

benthos-umh extends Benthos with 13 specialized plugins organized by function:

### Input Plugins
Industrial protocol connectors that read data from field devices:
- `opcua_plugin/` - OPC UA protocol input/output (Server/Client)
- `modbus_plugin/` - Modbus TCP/RTU protocol (Master/Slave)
- `s7comm_plugin/` - Siemens S7 protocol (S7-300/400/1200/1500)
- `sparkplug_plugin/` - Sparkplug B MQTT protocol (Edge/Host)
- `eip_plugin/` - Ethernet/IP protocol (Allen-Bradley)
- `sensorconnect_plugin/` - ifm IO-Link Master integration

**Integration**: Used in Bridge "Read" flows, orchestrated by UMH Core FSM

### Processing Plugins
Data transformation and enrichment processors:
- `tag_processor_plugin/` - Individual sensor data processing (metadata injection, value wrapping)
- `stream_processor_plugin/` - Multi-source data aggregation with state management
- `downsampler_plugin/` - Data reduction and aggregation (time-based sampling)
- `topic_browser_plugin/` - Topic discovery and metadata extraction
- `classic_to_core_plugin/` - UMH Classic to Core migration transformer
- `nodered_js_plugin/` - Node-RED JavaScript execution engine (goja-based)

**Integration**: Used in `pipeline.processors[]` section of DFC YAML

### Output Plugin
Unified Namespace writer with validation:
- `uns_plugin/` - Unified Namespace output with topic validation and Kafka batching

**Integration**: Only works within UMH Core (requires embedded Redpanda)

### Plugin Registration

All plugins auto-register during `init()`:
- Input plugins: `input.RegisterPlugin()`
- Processor plugins: `processor.RegisterPlugin()`
- Output plugins: `output.RegisterPlugin()`

**Discovery**: Run `benthos list inputs`, `benthos list processors`, `benthos list outputs`

## Key Concepts

- **Data Flow Component (DFC)**: YAML-defined data pipeline configuration
- **Protocol Converter**: Bridge between industrial protocols and UNS
- **Stream Processor**: Aggregates multiple data sources into unified streams
- **Tag Processor**: Handles individual sensor/tag data points
- **UNS (Unified Namespace)**: Central data structure following ISA-95 standard
- **Topic Browser**: Discovers and catalogs available data topics
- **Downsampler**: Reduces data volume while preserving important changes

## Integration with UMH Core

benthos-umh is **orchestrated by UMH Core** - it does not run standalone. UMH Core manages benthos instances through FSM state machines and S6 process supervision.

**High-level flow**:
1. User deploys bridge in ManagementConsole
2. umh-core Agent generates benthos config from templates
3. umh-core launches benthos process via S6
4. benthos process runs and sends data to Kafka

**For orchestration details** (FSM, S6, template expansion, logging, debugging in production), see `umh-core/CLAUDE.md`.

## Debugging benthos

**Config validation errors**:
```bash
# Test config without running
benthos lint -c config.yaml

# Check for template expansion issues
grep "{{" config.yaml  # Should be empty if variables expanded
```

**Plugin errors**:
```bash
# List available plugins
benthos list inputs
benthos list processors
benthos list outputs

# Run with debug logging
benthos run -c config.yaml --log.level DEBUG
```

**Connection issues**:
- Check protocol-specific logs for timeout/refused/unreachable errors
- Verify input plugin addresses, ports, credentials
- Test connectivity outside benthos (ping, telnet, protocol-specific tools)

**Data processing issues**:
- Enable debug logging to see message flow
- Check processor JavaScript for syntax errors (goja engine)
- Verify metadata fields are set correctly (msg.meta.*)
- Test with minimal config to isolate the problem

**For production debugging in UMH Core environment**, see `umh-core/CLAUDE.md`.

## Message Architecture

benthos-umh uses **Node-RED style message format** with separate payload and metadata:

```javascript
{
  "payload": {
    "value": 42,
    "timestamp_ms": 1730986400000
  },
  "meta": {
    "location_path": "enterprise.site.area.line",
    "data_contract": "_raw",
    "tag_name": "temperature",
    "virtual_path": "motor.electrical",  // Optional
    "umh_topic": "umh.v1.enterprise.site.area.line._raw.motor.electrical.temperature"  // Auto-generated
  }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [united-manufacturing-hub/benthos-umh](https://github.com/united-manufacturing-hub/benthos-umh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
