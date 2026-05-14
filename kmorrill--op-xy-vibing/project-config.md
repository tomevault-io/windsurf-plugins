---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants (Claude, ChatGPT, Codex, etc.) helping users with the OP‑XY Vibe Coding system.
---

# Claude AI Assistant Guide for OP‑XY Vibe Coding

This document provides comprehensive guidance for AI assistants (Claude, ChatGPT, Codex, etc.) helping users with the OP‑XY Vibe Coding system.

## Quick Reference for First-Time Users

When a user asks you to help them get started with this system, follow this sequence:

### 1. First-Run Server Setup

**Always start by cleaning up any existing processes:**
```bash
# Kill any hanging processes
make kill-procs
make kill-ws-port

# Send MIDI panic if device is connected
make panic PORT="OP-XY" || echo "No OP-XY connected yet"
```

**Check dependencies and set up environment:**
```bash
# Check if Python is available
python3 --version

# Create virtual environment if it doesn't exist
if [ ! -d "venv" ]; then
    python3 -m venv venv
fi

# Activate and install dependencies
source venv/bin/activate
pip install -r requirements.txt

# Verify MIDI setup
python3 -c "import mido; print('MIDI ports:', mido.get_output_names())"
```

**Start the server:**
```bash
# From the project directory, with venv activated:
source venv/bin/activate
python3 -m conductor.conductor_server --loop loop.json --port "OP-XY"
```

**Expected output:**
```
[http] serving UI on http://127.0.0.1:8080  
[ws] Conductor listening on ws://127.0.0.1:8765
```

**Validate the setup:**
```bash
# Test HTTP server
curl -s http://127.0.0.1:8080/ | head -5

# Should return HTML starting with <!doctype html>
```

### 2. Common Server Management

**Server health check:**
```bash
# HTTP endpoint test
curl -s http://127.0.0.1:8080/ >/dev/null && echo "HTTP server OK"

# Process check
ps aux | grep conductor_server | grep -v grep || echo "Server not running"
```

**Restart the server:**
```bash
make kill-procs
make kill-ws-port
source venv/bin/activate
python3 -m conductor.conductor_server --loop loop.json --port "OP-XY"
```

## Understanding the System Architecture

### Core Components

1. **loop.json** - Single source of truth for the musical loop
2. **Conductor server** - Manages the JSON, validates changes, serves UI
3. **Playback engine** - Converts JSON to real-time MIDI events  
4. **Web UI** - Browser-based editor at http://127.0.0.1:8080
5. **WebSocket API** - Real-time communication at ws://127.0.0.1:8765

### Key Principles (from AGENTS.md)

- **Single source of truth:** All edits go through loop.json
- **Concurrency via docVersion:** Every change increments docVersion
- **Minimal, reversible edits:** Use JSON Patch operations when possible
- **No look-ahead scheduling:** Events scheduled only on current tick
- **Human in the loop:** Ask for clarification on ambiguous requests

## Working with loop.json

### Format Overview

The loop.json file uses the `opxyloop-1.0` schema. Key structure:

```json
{
  "version": "opxyloop-1.0",
  "meta": {
    "tempo": 120,
    "ppq": 96, 
    "stepsPerBar": 16
  },
  "tracks": [
    {
      "id": "drums-foundation",
      "type": "sampler",
      "midiChannel": 0,
      "role": "drums",
      "pattern": {
        "lengthBars": 2,
        "steps": [...]
      }
    }
  ],
  "docVersion": 1
}
```

### Safe Editing Practices

**Always validate before editing:**
```bash
# Validate current loop
make validate FILE=loop.json

# Validate all test fixtures  
make validate-fixtures
```

**Use atomic writes:**
- Never edit loop.json directly while server is running
- Use the WebSocket API or JSON Patch operations
- The server handles atomic temp-file writes automatically

## MIDI Device Safety

### Essential Safety Measures

**Before starting any session:**
```bash
# Always send panic first
make panic PORT="OP-XY"

# Kill any existing processes
make kill-procs
```

**MIDI Channel Guidelines:**
- Drums typically use channel 0 (zero-based) 
- Bass, chords, melody use channels 1, 2, 3, etc.
- Channel 9 reserved for GM drum kits
- MIDI values must be 0-127 (notes, velocities, CC values)

**Note Lifecycle:**
- Every Note On must have a corresponding Note Off
- Server maintains active-notes ledger
- All Notes Off sent on stop/disconnect/panic

## Troubleshooting Guide

### Server Won't Start

```bash
# Check if port is in use
lsof -i :8765 || echo "Port 8765 is free"
lsof -i :8080 || echo "Port 8080 is free"

# Check Python environment
source venv/bin/activate
python3 -c "import mido, websockets, jsonpatch; print('Dependencies OK')"

# Check file permissions
ls -la loop.json
```

### MIDI Connection Issues

```bash
# List available MIDI ports
python3 -c "import mido; print(mido.get_output_names())"

# Test basic MIDI output
python3 -c "
import mido
out = mido.open_output('OP-XY')
out.send(mido.Message('note_on', note=60, velocity=64))
out.send(mido.Message('note_off', note=60))
out.close()
print('MIDI test sent')
"
```

### Loop JSON Issues

```bash
# Validate and get detailed errors
make validate FILE=loop.json

# Check JSON syntax
python3 -c "import json; json.load(open('loop.json')); print('Valid JSON')"

# View loop structure
python3 -c "
import json
loop = json.load(open('loop.json'))
print(f'Tempo: {loop['meta']['tempo']} BPM')
print(f'Tracks: {len(loop['tracks'])}') 
print(f'Doc version: {loop.get('docVersion', 'missing')}')
"
```

## Testing and Validation

### Quick Tests

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kmorrill/op-xy-vibing](https://github.com/kmorrill/op-xy-vibing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
