---
trigger: always_on
description: **AetherLens Home Edition** is an open-source cost and usage monitoring platform for home labs, smart homes, IoT
---

# CLAUDE.md - AI Assistant Development Guidelines

## Project Context

**AetherLens Home Edition** is an open-source cost and usage monitoring platform for home labs, smart homes, IoT
devices, and personal cloud services. This document provides context and guidelines for AI assistants (particularly
Claude and other LLMs) when contributing to this project.

## Project Philosophy

- **Community First**: Built by home lab enthusiasts, for home lab enthusiasts
- **Privacy by Default**: All data stays local unless explicitly configured otherwise
- **Security First**: Defense in depth, encrypted credentials, minimal attack surface
- **Simplicity Over Features**: Start minimal, iterate based on real user needs
- **Plugin-Driven**: Core is minimal; functionality comes from community plugins
- **Resource-Efficient**: Must run on Raspberry Pi 4 or equivalent (\<2GB RAM)
- **Observable**: Comprehensive logging and metrics for debugging
- **Learning Platform**: Great for showcasing full-stack engineering skills

## Target Audience

### Primary Users

- **Home Lab Enthusiasts** - Running Proxmox, TrueNAS, or Kubernetes clusters
- **Smart Home Power Users** - Complex Home Assistant or Node-RED setups
- **Self-Hosters** - Managing their own services (Nextcloud, Plex, etc.)
- **Crypto Miners** - Tracking profitability vs. electricity costs
- **Solar Power Owners** - Optimizing self-consumption
- **Remote Workers** - Managing home office expenses

### Secondary Users

- **Small Businesses** - Single-location monitoring
- **Apartment Dwellers** - Understanding energy costs
- **Students** - Learning about energy efficiency

## Code Standards

### Language Preferences

**Core Engine**: Python 3.11+ (FastAPI)

- Prioritize readability and AI-assisted development
- Use type hints extensively
- Async/await for all I/O operations
- Optimize hot paths in Rust only when profiling shows bottlenecks

**Plugin SDK**: Python (primary)

- Focus on single language for v1.0
- Go/Rust/TypeScript support in future phases
- Keep plugin API simple and well-documented

**Frontend**: TypeScript + React 18+

- Vite for build tooling
- Tailwind CSS for styling
- Recharts for data visualization
- Functional components with hooks

### Code Style

```python
# Python: Type hints, descriptive names, async patterns
from typing import List, Dict, Optional
from dataclasses import dataclass
import asyncio

@dataclass
class Metric:
    """Represents a single metric data point"""
    device_id: str
    timestamp: float
    metric_type: str
    value: float
    unit: str
    tags: Optional[Dict[str, str]] = None

async def collect_device_metrics(device_id: str) -> List[Metric]:
    """
    Collect metrics from a device asynchronously.
    
    Args:
        device_id: Unique identifier for the device
        
    Returns:
        List of collected metrics
        
    Raises:
        DeviceOfflineError: If device is not reachable
        TimeoutError: If collection takes too long
    """
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(f"http://{device_id}/status", timeout=5) as resp:
                data = await resp.json()
                return parse_metrics(data)
    except asyncio.TimeoutError:
        logger.error(f"Timeout collecting from {device_id}")
        raise
```

```typescript
// TypeScript: Explicit types, functional style
interface DeviceMetric {
  timestamp: number;
  deviceId: string;
  powerWatts: number;
  costPerHour: number;
}

interface CostSummary {
  totalKwh: number;
  totalCost: number;
  currency: string;
}

// Prefer const and arrow functions
const calculateDailyCost = (metrics: DeviceMetric[]): number => {
  return metrics.reduce((sum, m) => sum + (m.costPerHour * 24), 0);
};

// Use async/await for API calls
const fetchCurrentMetrics = async (): Promise<DeviceMetric[]> => {
  const response = await fetch('/api/v1/metrics/current');
  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }
  return response.json();
};
```

### Error Handling

Always include context in error messages and make them actionable:

```python
# ✅ Good - Actionable error with context
raise DeviceOfflineError(
    f"Cannot connect to device '{device_id}' at {ip_address}. "
    f"Please check that the device is powered on and connected to the network."
)

# ❌ Bad - Vague error
raise Exception("Connection failed")
```

Never use bare `except:` clauses:

```python
# ✅ Good
try:
    result = await fetch_data()
except (ConnectionError, TimeoutError) as e:
    logger.error(f"Network error: {e}")
    return None
except ValueError as e:
    logger.error(f"Invalid data: {e}")
    raise

# ❌ Bad
try:
    result = await fetch_data()
except:
    pass
```

## Architecture Principles

### Data Flow

```
Device/Service → Plugin → Collector → Processor → Storage → API → UI/Export
                   ↓         ↓          ↓           ↓
                 Isolate   Buffer    Enrich      Persist
```

1. **Collection**: Plugins gather metrics from devices/services
1. **Processing**: Normalize, enrich, and calculate derived metrics
1. **Storage**: Persist to TimescaleDB with retention policies
1. **Serving**: API layer serves processed data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dreffed/aetherlens](https://github.com/Dreffed/aetherlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
