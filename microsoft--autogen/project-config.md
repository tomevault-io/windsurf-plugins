---
trigger: always_on
description: AutoGen is a multi-language framework for creating AI agents that can act autonomously or work alongside humans. The project has separate Python and .NET implementations with their own development workflows.
---

# AutoGen Multi-Agent AI Framework

AutoGen is a multi-language framework for creating AI agents that can act autonomously or work alongside humans. The project has separate Python and .NET implementations with their own development workflows.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Environment Setup

**CRITICAL**: Install both .NET 8.0 and 9.0 for full compatibility:
- Install uv package manager: `python3 -m pip install uv` 
- Install .NET 9.0 SDK: `wget https://dot.net/v1/dotnet-install.sh && chmod +x dotnet-install.sh && ./dotnet-install.sh --channel 9.0`
- Install .NET 8.0 runtime: `./dotnet-install.sh --channel 8.0 --runtime dotnet && ./dotnet-install.sh --channel 8.0 --runtime aspnetcore`
- Update PATH: `export PATH="$HOME/.dotnet:$PATH"`

### Python Development Workflow

**Bootstrap and build Python environment:**
```bash
cd /home/runner/work/autogen/autogen/python
uv sync --all-extras  # NEVER CANCEL: Takes 2 minutes. Set timeout to 300+ seconds.
source .venv/bin/activate
```

**Validate Python development:**
```bash
# Quick validation (under 1 second each)
poe format  # Code formatting
poe lint    # Linting with ruff

# Type checking - NEVER CANCEL these commands
poe mypy     # Takes 6 minutes. Set timeout to 600+ seconds.
poe pyright  # Takes 41 seconds. Set timeout to 120+ seconds.

# Individual package testing (core package example)
poe --directory ./packages/autogen-core test  # Takes 10 seconds. Set timeout to 60+ seconds.

# Documentation - NEVER CANCEL
poe docs-build  # Takes 1 minute 16 seconds. Set timeout to 300+ seconds.
```

**CRITICAL TIMING EXPECTATIONS:**
- **NEVER CANCEL**: Python environment setup takes 2 minutes minimum
- **NEVER CANCEL**: mypy type checking takes 6 minutes 
- **NEVER CANCEL**: Documentation build takes 1+ minutes
- Format/lint tasks complete in under 1 second
- Individual package tests typically complete in 10-60 seconds

### .NET Development Workflow

**Bootstrap and build .NET environment:**
```bash
cd /home/runner/work/autogen/autogen/dotnet
export PATH="$HOME/.dotnet:$PATH"
dotnet restore  # NEVER CANCEL: Takes 53 seconds. Set timeout to 300+ seconds.
dotnet build --configuration Release  # NEVER CANCEL: Takes 53 seconds. Set timeout to 300+ seconds.
```

**Validate .NET development:**
```bash
# Unit tests - NEVER CANCEL
dotnet test --configuration Release --filter "Category=UnitV2" --no-build  # Takes 25 seconds. Set timeout to 120+ seconds.

# Format check (if build fails) 
dotnet format --verify-no-changes

# Run samples
cd samples/Hello
dotnet run
```

**CRITICAL TIMING EXPECTATIONS:**
- **NEVER CANCEL**: .NET restore takes 53 seconds minimum
- **NEVER CANCEL**: .NET build takes 53 seconds minimum  
- **NEVER CANCEL**: .NET unit tests take 25 seconds minimum
- All build and test commands require appropriate timeouts

### Complete Validation Workflow

**Run full check suite (Python):**
```bash
cd /home/runner/work/autogen/autogen/python
source .venv/bin/activate
poe check  # NEVER CANCEL: Runs all checks. Takes 7+ minutes total. Set timeout to 900+ seconds.
```

## Validation Scenarios

### Manual Validation Requirements
Always manually validate changes by running complete user scenarios after making modifications:

**Python validation scenarios:**
1. **Import test**: Verify core imports work:
   ```python
   from autogen_agentchat.agents import AssistantAgent
   from autogen_core import AgentRuntime
   from autogen_ext.models.openai import OpenAIChatCompletionClient
   ```

2. **AutoGen Studio test**: Verify web interface can start:
   ```bash
   autogenstudio ui --help  # Should show help without errors
   ```

3. **Documentation test**: Build and verify docs generate without errors:
   ```bash
   poe docs-build && ls docs/build/index.html
   ```

**.NET validation scenarios:**
1. **Sample execution**: Run Hello sample to verify runtime works:
   ```bash
   cd dotnet/samples/Hello && dotnet run --help
   ```

2. **Build validation**: Ensure all projects compile:
   ```bash
   dotnet build --configuration Release --no-restore
   ```

3. **Test execution**: Run unit tests to verify functionality:
   ```bash
   dotnet test --filter "Category=UnitV2" --configuration Release --no-build
   ```

## Common Issues and Workarounds

### Network-Related Issues
- **Python tests may fail** with network errors (tiktoken downloads, Playwright browser downloads) in sandboxed environments - this is expected
- **Documentation intersphinx warnings** due to inability to reach external documentation sites - this is expected
- **Individual package tests work better** than full test suite in network-restricted environments

### .NET Runtime Issues  
- **Requires both .NET 8.0 and 9.0**: Build uses 9.0 SDK but tests need 8.0 runtime
- **Global.json specifies 9.0.100**: Must install exact .NET 9.0 version or later
- **Path configuration critical**: Ensure `$HOME/.dotnet` is in PATH before system .NET

### Python Package Issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/autogen](https://github.com/microsoft/autogen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
