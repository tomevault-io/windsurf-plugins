---
trigger: always_on
description: The h4cker repository is a comprehensive collection of cybersecurity references, scripts, tools, code, and educational resources. It serves as supplemental material for cybersecurity books, video courses, and live training created by Omar Santos.
---

# H4cker Cybersecurity Resources Repository

The h4cker repository is a comprehensive collection of cybersecurity references, scripts, tools, code, and educational resources. It serves as supplemental material for cybersecurity books, video courses, and live training created by Omar Santos.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Environment Setup
- Python 3.12+ is available and working
- Node.js 20.19+ is available and working  
- Docker 28.0+ is available and working
- Repository is a collection of cybersecurity resources, NOT a traditional application that builds into a single artifact

### Core Validation Commands
- Link checking (primary CI validation): `lychee --config lychee.toml --no-progress --verbose './**/*.md'` -- takes 15-30 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
- Install link checker if missing: `curl -L https://github.com/lycheeverse/lychee/releases/latest/download/lychee-x86_64-unknown-linux-gnu.tar.gz | tar -xz && sudo mv lychee /usr/local/bin/`

### Python Scripts and Tools Validation  
- Install OSINT tool dependencies: `cd osint/quick_recon && pip3 install -r requirements.txt` -- takes 10-20 seconds. NEVER CANCEL.
- Install common dependencies: `pip3 install python-nmap requests termcolor colorama beautifulsoup4` -- takes 15-30 seconds. NEVER CANCEL.
- Install system tools as needed: `sudo apt-get update && sudo apt-get install -y nmap` -- takes 60-120 seconds. NEVER CANCEL. Set timeout to 180+ seconds.

### Docker Security Examples
- Build Docker security examples: `cd docker-and-k8s-security/docker && make build-naive` -- takes 20-30 seconds for first build. NEVER CANCEL. Set timeout to 300+ seconds for initial builds.
- Other Docker targets: `make build-non-root`, `make build-distroless`, `make run`
- Docker builds may take up to 5 minutes on first run due to image downloads. NEVER CANCEL.

## Validation Scenarios

### Repository Health Check
Always run these commands to validate repository state:
1. `lychee --config lychee.toml --no-progress --verbose './README.md'` -- Test core documentation links
2. `cd osint/quick_recon && pip3 install -r requirements.txt` -- Verify Python dependencies
3. `cd docker-and-k8s-security/docker && make build-naive` -- Test Docker functionality

### Script Testing Scenarios  
- Test Python scripts individually as most require specific network access or targets
- Many scripts are designed for penetration testing environments and may not run fully in restricted environments
- Focus on syntax validation and dependency checking rather than full execution
- Example: `python3 -m py_compile programming-and-scripting-for-cybersecurity/recon_scripts/scanning/basic_ping_sweep.py`

### Documentation Updates
- Always run link checking after modifying markdown files: `lychee --config lychee.toml --no-progress --verbose './**/*.md'`
- Links to external sites may fail due to network restrictions - this is expected
- Focus on internal repository links and file references

## Repository Structure Overview

### Key Directories
- `programming-and-scripting-for-cybersecurity/` - Educational scripts in Python, Bash, etc.
- `osint/` - Open Source Intelligence tools and resources
- `docker-and-k8s-security/` - Container security examples and tools
- `threat-hunting/` - Threat hunting resources and techniques
- `exploit-development/` - Exploit development resources
- `web-application-testing/` - Web security testing resources
- `dfir/` - Digital Forensics and Incident Response materials
- `cheat-sheets/` - Quick reference guides

### Working with Individual Tools
- Each directory contains specialized tools for different cybersecurity domains
- Tools are meant to be used individually rather than as part of a larger application
- Many tools require network access to external services (may not work in restricted environments)
- Focus on code review, syntax checking, and educational value rather than live execution

## Common Tasks

### Repository Health Check (Complete Validation Scenario)
Run this complete validation sequence after making changes:
```bash
cd /home/runner/work/h4cker/h4cker
echo "1. Testing link checker..."
lychee --config lychee.toml --no-progress --verbose './README.md' 
echo "2. Testing Python dependencies..."
cd osint/quick_recon && pip3 install -r requirements.txt
echo "3. Testing Docker functionality..."
cd /home/runner/work/h4cker/h4cker/docker-and-k8s-security/docker && make build-naive
```
Expected results: Links mostly pass (external sites may fail), Python deps install successfully, Docker builds complete.

### Adding New Resources
- Place new scripts in appropriate domain directories (`programming-and-scripting-for-cybersecurity/`, `osint/`, etc.)
- Update documentation to reference new resources
- Run link checking to validate any new markdown content: `lychee --config lychee.toml --no-progress --verbose './**/*.md'`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [The-Art-of-Hacking/h4cker](https://github.com/The-Art-of-Hacking/h4cker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
