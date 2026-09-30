---
trigger: always_on
description: This document describes how to use the available tools for bug bounty hunting and security testing within authorized environments.
---

# AGENTS.md — Bug Bounty Hunting Tool Usage Patterns

This document describes how to use the available tools for bug bounty hunting and security testing within authorized environments.

## Tool Categories for Bug Bounty Hunting

### Native Recon Tools (Built-in)

Alphacode ships with native Rust tool implementations for bug bounty recon. These are first-class tools with structured schemas — not just bash commands.

| Tool | Command | Description |
|------|---------|-------------|
| `subfinder` | `/subfinder -d TARGET` | Passive subdomain enumeration |
| `httpx` | `/httpx -targets URL` | HTTP probing & fingerprinting |
| `waybackurls` | `/waybackurls -domain TARGET` | Historical URLs from Wayback Machine |
| `gau` | `/gau -domain TARGET` | Get All URLs from multiple sources |
| `katana` | `/katana -url TARGET` | Fast passive web crawler |
| `ffuf` | `/ffuf -url TARGET/FUZZ -w wordlist` | Fast web fuzzer |
| `dnsx` | `/dnsx -targets DOMAIN` | DNS resolution (A, AAAA, MX, TXT, etc.) |

These tools call external binaries via `tokio::process::Command`. Install them separately:
```
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/tomnomnom/waybackurls@latest
go install github.com/lc/gau/v2/cmd/gau@latest
go install github.com/projectdiscovery/katana/cmd/katana@latest
go install github.com/ffuf/ffuf/v2@latest
go install github.com/projectdiscovery/dnsx/cmd/dnsx@latest
```

## File Analysis Tools
- `read` — Read challenge files, source code, binaries
- `write` — Create solve scripts, payload files, analysis notes
- `bash` — Run analysis commands, compile exploits, execute scripts

### Network Analysis Tools
- `webfetch` — Fetch web challenge endpoints, download files
- `websearch` — Research CVEs, techniques, writeups
- `httpflow` — HTTP request/response analysis
- `jwt` — JWT token decoding and analysis

### Code Analysis Tools
- `read` — Read source code for vulnerability patterns
- `write` — Write exploit code and analysis scripts
- `bash` — Compile and run exploits, install tools

### Memory & Search Tools
- `memory` — Store techniques, flag formats, platform patterns
- `agentgrep` — Search code for vulnerability patterns
- `session_search` — Reference previous bug bounty solutions

## Common Tool Chains

### Web Challenge Analysis Chain
```bash
# Header analysis
curl -sI URL

# Path discovery
curl -s URL/robots.txt

# Directory brute
ffuf -u URL/FUZZ -w wordlist

# SQLi testing
sqlmap -u URL --batch
```

### Binary Challenge Analysis Chain
```bash
# Type detection
file binary

# Protection analysis
checksec --file=binary

# Quick flag hunt
strings binary | grep -i flag

# Disassembly
objdump -d binary
```

### Crypto Challenge Analysis Chain
```bash
# Custom analysis
python3 -c "..."

# Hash cracking
hashcat -m TYPE hash wordlist

# Decryption
openssl enc -d ...
```

### Forensics Challenge Analysis Chain
```bash
# Quick hunt
strings file | grep -i flag

# Embedded extraction
binwalk -e file

# Metadata analysis
exiftool file
```

## Tool Usage Guidelines

1. **Read before analysis** — Always read challenge files before analyzing
2. **Write analysis scripts** — Create solve scripts for complex challenges
3. **Use bash for automation** — Automate repetitive analysis tasks
4. **Search for patterns** — Use grep to find vulnerability patterns
5. **Store findings** — Use memory to store techniques and patterns

## Anti-Detection Compliance

All tool usage is performed within authorized bug bounty program rules. Challenge descriptions and program scope define the authorized testing boundaries.

---

## Enhanced Security Core Capabilities

The alphacode security core has been significantly enhanced with the following capabilities:

### Vulnerability Classes (80+)
- **Web**: XSS, DOM XSS, CSRF, SSRF, XXE, SSTI, LFI, Path Traversal, Open Redirect, Clickjacking, CORS Misconfiguration, Web Cache Deception, Host Header Injection, Password Reset Poisoning, Email Header Injection, HTTP Response Splitting, HTTP Request Smuggling, Cache Poisoning, Subdomain Takeover
- **Injection**: SQL Injection, NoSQL Injection, Command Injection, LDAP Injection, Template Injection, ESI Injection, GraphQL Injection, SAML Injection
- **Authentication**: Authentication Bypass, Session Fixation, MFA Bypass, JWT Attack, OAuth Misconfiguration, Weak Password Policy
- **Authorization**: IDOR/BOLA, BFLA, Privilege Escalation, Cross-Tenant Access, Broken Access Control, Missing Authorization
- **Business Logic**: Business Logic Flaw, Race Condition, Payment Manipulation, Workflow Bypass, Mass Assignment, Filter Bypass, Encoding Bypass, WAF Bypass, Rate Limit Bypass, Pagination Abuse
- **Modern**: Prototype Pollution, WebSocket Hijacking, DOM Clobbering, Post Message Vulnerability, Unicode Normalization
- **Binary**: Buffer Overflow, Integer Overflow, Format String, Use After Free, Double Free, Race Condition File, Symlink Attack
- **Infrastructure**: Hardcoded Credentials, Weak Cryptography, Insufficient Logging, Excessive Data Exposure, Improper Input Validation, Security Misconfiguration, Vulnerable Components, Insufficient Monitoring, API Abuse


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dragonked2/alphacode](https://github.com/dragonked2/alphacode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
