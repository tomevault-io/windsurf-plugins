---
trigger: always_on
description: The Bazzite Gaming Optimization Suite is a comprehensive, enterprise-grade gaming optimization framework centered around the `bazzite-optimizer.py` master script (7,637 lines, 300KB) with a modern GTK4 graphical interface, complete ML/AI engine, and React Native mobile app. This production system delivers **15-25% performance improvements** for high-end gaming configurations.
---

# Copilot Instructions - Bazzite Gaming Optimization Suite

## Project Overview

The Bazzite Gaming Optimization Suite is a comprehensive, enterprise-grade gaming optimization framework centered around the `bazzite-optimizer.py` master script (7,637 lines, 300KB) with a modern GTK4 graphical interface, complete ML/AI engine, and React Native mobile app. This production system delivers **15-25% performance improvements** for high-end gaming configurations.

**Target Platform**: Bazzite Linux (Fedora-based immutable gaming OS)  
**Primary Hardware**: NVIDIA RTX/AMD RDNA2-3 GPUs, Intel/AMD CPUs, Steam Deck, ROG Ally  
**License**: MIT  
**Current Version**: 1.6.0  
**Codebase**: 34,000+ lines across 84 Python/TypeScript files

## Architecture & Core Components

### Master Script
- **bazzite-optimizer.py** (7,637 lines): Primary entrypoint with 16 specialized optimizer classes
  - Profile-based optimization (Competitive, Balanced, Streaming, Safe Defaults)
  - Intelligent system tuning with validation and rollback capabilities
  - Transaction handling with 100% validation success rate

### Tools Suite
- **gaming-manager-suite.py**: System control and gaming mode management
- **gaming-monitor-suite.py**: Real-time performance metrics with curses dashboard
- **gaming-maintenance-suite.sh**: Automated benchmarking and maintenance
- **undo_bazzite-optimizer.py**: Configuration restore utility

### Advanced Features
- **GUI** (gui/): GTK4 graphical interface (~2,600 lines)
- **ML Engine** (ml_engine/): Machine learning optimizer (~7,300 lines)
  - Real-time data collection (benchmark_collector.py)
  - Model optimization (model_optimizer.py)
- **AI Engine** (ai_engine/): DQN reinforcement learning (dqn_agent.py, 406 lines)
- **Mobile App** (mobile-app/): React Native companion app (~1,200 lines)
- **Mobile API** (mobile_api/): FastAPI WebSocket server (websocket_server.py, 405 lines)

## Technology Stack

### Core Technologies
- **Python 3.8+**: Primary language for all optimization logic
- **Bash**: Shell scripts for system-level operations
- **GTK4**: Modern graphical interface
- **PyTorch**: Deep learning and reinforcement learning
- **React Native**: Mobile companion application
- **FastAPI**: WebSocket server for mobile integration

### Key Dependencies
- `psutil`: System metrics collection
- `stress-ng`, `sysbench`: Benchmarking tools
- `rpm-ostree`: Immutable OS management (Bazzite-specific)
- `ujust`: Bazzite system commands integration
- `system76-scheduler`, `GameMode`: Gaming optimizations

## Code Style & Conventions

### Python Style
- **PEP 8** with 88-character line limit (Black formatter)
- **Type hints** required for all public function parameters and returns
- **Docstrings** mandatory for all classes and public methods (Google/NumPy style)
- **Exception handling**: Graceful with specific error messages

Example:
```python
def collect_metrics(self, interval: int = 2) -> Dict[str, float]:
    """
    Collect system performance metrics.
    
    Args:
        interval: Sampling interval in seconds
        
    Returns:
        Dictionary containing CPU, GPU, memory metrics
        
    Raises:
        SystemError: If metrics collection fails
    """
    try:
        # Implementation
        pass
    except Exception as e:
        raise SystemError(f"Metrics collection failed: {e}")
```

### Shell Script Style
- **Bash shebang**: `#!/bin/bash`
- **Strict mode**: `set -euo pipefail`
- **Quote variables**: Always use `"$variable"`
- **Functions** for code reuse
- **Error handling** with meaningful messages

### Naming Conventions
- **Scripts**: hyphenated (e.g., `bazzite-optimizer.py`, `gaming-manager-suite.py`)
- **Modules/Functions**: snake_case (e.g., `collect_metrics`, `apply_profile`)
- **Classes**: PascalCase (e.g., `GamingModeController`, `MetricsCollector`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `DEFAULT_INTERVAL`, `MAX_RETRIES`)

### Color-Coded Output
Consistent `Colors` class used across all tools:
```python
class Colors:
    GREEN = '\033[92m'
    YELLOW = '\033[93m'
    RED = '\033[91m'
    BLUE = '\033[94m'
    CYAN = '\033[96m'
    BOLD = '\033[1m'
    RESET = '\033[0m'
```

## Development Commands

### Prerequisites (Bazzite Linux)
```bash
# System dependencies
sudo dnf install python3-psutil python3-configparser stress-ng sysbench

# Development tools
sudo dnf install python3-pip python3-devel
pip3 install --user black isort flake8 mypy pylint
```

### Environment Setup
```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate

# Install development dependencies
pip install -e .[dev]

# Make scripts executable
chmod +x bazzite-optimizer.py gaming-manager-suite.py gaming-monitor-suite.py gaming-maintenance-suite.sh
```

### Code Quality
```bash
# Format code
black .
isort .

# Lint
flake8

# Type checking
mypy bazzite-optimizer.py
```

### Testing
```bash
# Run all tests
pytest -q

# Run specific test
pytest tests/test_kernel_param_fix.py -v

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doublegate/Bazzite-Config](https://github.com/doublegate/Bazzite-Config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
