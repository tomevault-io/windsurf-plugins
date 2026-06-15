---
trigger: always_on
description: Generates matching and non matching examples.
---

# AGENT.md

# Dex

Dex is a single binary developer command center built in Go.

It is not a basic utility collection. It is an interactive terminal operating layer for developers that combines networking, process intelligence, API tooling, JSON tooling, system diagnostics, regex engineering, benchmarking, files, clipboard, and terminal visualization.

Dex must feel fast, beautiful, local first, deeply useful, and powerful enough that developers install it on every machine.

## Core Principles

1. One binary.
2. No background daemon required by default.
3. Every command works in plain CLI mode.
4. Every major module also has an interactive TUI mode.
5. Output must support human, JSON, CSV, Markdown, and raw modes.
6. No AI features for now.
7. No logs module.
8. No debug module.
9. No dependency manager, pip environment, git, or code context features.
10. The CLI should feel premium, polished, and visually alive.

## Global Command Shape

```bash
dex <module> <command> [args] [flags]
```

Examples:

```bash
dex network ip
dex network ports
dex process inspect 1234
dex api replay session.dexapi
dex json lens data.json
dex system health
dex regex lab
```

## Global Flags

```bash
--json
--csv
--markdown
--raw
--watch
--interval 1s
--no-color
--theme dark
--profile default
--save
--export path
--copy
--quiet
--verbose
```

## Global Interactive Mode

```bash
dex
```

Launches the Dex dashboard.

Sections:

```text
Network
Processes
API
JSON
System
Regex
Benchmark
Files
Clipboard
Terminal
Settings
```

The TUI must support:

```text
vim navigation
fuzzy search
command palette
split panes
live charts
copy selected value
export current view
mouse support
keyboard first workflow
theme picker
session restore
```

Recommended Go stack:

```text
cobra
bubbletea
lipgloss
bubbles
sqlite
badger or boltdb
net/http
net
os/exec
gopsutil
regexp/syntax
encoding/json
```

# Module: Network

The network module should be the deepest part of Dex.

## Basic Identity

```bash
dex network ip
dex network ip --all
dex network public-ip
dex network interfaces
dex network interface en0
dex network mac
dex network hostname
dex network gateway
dex network routes
dex network dns-config
dex network proxy
dex network vpn
```

## DNS

```bash
dex network dns google.com
dex network dns google.com --type A
dex network dns google.com --type AAAA
dex network dns google.com --type MX
dex network dns google.com --type TXT
dex network dns google.com --type CNAME
dex network dns google.com --type NS
dex network dns google.com --resolver 1.1.1.1
dex network dns-trace google.com
dex network dns-bench google.com
dex network dns-compare google.com
dex network dns-cache
dex network dns-flush
dex network dns-watch google.com
```

Novel DNS features:

```bash
dex network dns-story google.com
```

Shows the full resolution path, resolver used, TTLs, record changes, and suspicious mismatches.

```bash
dex network dns-drift google.com
```

Watches DNS over time and reports record changes.

## Ports

```bash
dex network ports
dex network ports --listening
dex network ports --open
dex network ports --range 3000-9000
dex network port 3000
dex network port kill 3000
dex network port owner 3000
dex network port free
dex network port suggest
dex network port reserve 3000
dex network port watch
```

Novel port features:

```bash
dex network port-timeline
```

Shows when ports opened, closed, and which process owned them.

```bash
dex network port-map
```

Visual map of local ports grouped by process.

## Connections

```bash
dex network connections
dex network connections --process node
dex network connections --remote
dex network connections --local
dex network connections --established
dex network connections --listening
dex network connections --country
dex network connections --process-tree
dex network connection inspect <id>
dex network connection kill <id>
dex network connections watch
```

Novel connection features:

```bash
dex network connection-radar
```

Live visual radar of outbound connections.

```bash
dex network process-map
```

Maps each process to its sockets, remote hosts, ports, protocols, and byte counts.

## Latency

```bash
dex network ping google.com
dex network latency google.com
dex network latency google.com --tcp 443
dex network latency google.com --http
dex network latency google.com --tls
dex network latency google.com --dns
dex network latency-bench google.com
dex network latency-watch google.com
dex network jitter google.com
```

Novel latency features:

```bash
dex network latency-stack google.com
```

Breaks latency into DNS, TCP, TLS, first byte, download, and total time.

```bash
dex network latency-compare google.com cloudflare.com openai.com
```

Shows side by side performance.

## Tracing

```bash
dex network traceroute google.com
dex network mtr google.com
dex network path google.com
dex network hops google.com
dex network route-to google.com
dex network path-watch google.com
```

Novel tracing features:

```bash
dex network path-map google.com
```

Draws an interactive route map in the terminal.

```bash
dex network route-drift google.com
```

Detects when network path changes over time.

## HTTP

```bash
dex network http https://example.com
dex network headers https://example.com
dex network status https://example.com

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [desenyon/dex](https://github.com/desenyon/dex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
