---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project does

pg_protoexport reads network capture files (.pcapng, .pcap), extracts PostgreSQL wire protocol conversations, and generates output in several formats: LaTeX diagrams (using the `bytefield` package), PQTrace-style tab-separated text, Markdown files with embedded Mermaid or PlantUML diagrams, and a self-contained "guided reading" HTML report.

It also performs **live capture** in-process via SharpPcap: `LiveCaptureSession` writes a `.pcapng` file from a live NIC for the duration of a workload, removing the need to start `tcpdump`/Wireshark by hand.

## Build and test commands

```bash
# Build entire solution
dotnet build pg_protoexport.slnx

# Build just the CLI
dotnet build pg_protoexport/pg_protoexport.csproj

# Run all tests
dotnet test pg_protoexport.tests/pg_protoexport.tests.csproj

# Run a single test by name
dotnet test pg_protoexport.tests/pg_protoexport.tests.csproj --filter "FullyQualifiedName~TestMethodName"

# Run the CLI. Port is now a `--port` option; if omitted, the dominant PostgreSQL
# port is auto-detected from each capture's TCP SYN handshake.
dotnet run --project pg_protoexport -- latex <file.pcapng> <output.tex>
dotnet run --project pg_protoexport -- latex <file.pcapng> <output.tex> --port 5432 --exact --row-bytes 32
dotnet run --project pg_protoexport -- pqtrace <file.pcapng> <output.txt>
dotnet run --project pg_protoexport -- mermaid <file.pcapng> sequenceDiagram <output.md>
dotnet run --project pg_protoexport -- mermaid <file.pcapng> packet <output.md>
dotnet run --project pg_protoexport -- plantuml <file.pcapng> sequenceDiagram <output.md>
dotnet run --project pg_protoexport -- plantuml <file.pcapng> packet <output.md>
dotnet run --project pg_protoexport -- html <file.pcapng> <output.html>
# ascii is a two-mode branch (capture_file comes before the mode keyword, like mermaid/plantuml).
# --console / --max-width are BRANCH options (declared on AsciiBranchSettings so they show in
# `ascii --help`), so they must be placed BEFORE the mode keyword, not after it.
dotnet run --project pg_protoexport -- ascii <file.pcapng> fields <output.txt>
dotnet run --project pg_protoexport -- ascii <file.pcapng> sequenceDiagram <output.txt>
dotnet run --project pg_protoexport -- ascii <file.pcapng> --console sequenceDiagram   # write to stdout, no file
dotnet run --project pg_protoexport -- ascii <file.pcapng> --console --max-width 120 fields

# Batch export: run every exporter + variant over every .pcapng in a directory.
# Produces <output-dir>/<input-stem>/capture.{tex,pqtrace.txt,ascii.txt,ascii.seq.txt,mermaid.{seq,pkt}.md,
# plantuml.{seq,pkt}.md,html} + capture_assets/ per input.
dotnet run --project pg_protoexport -- batchexport <input-dir>
dotnet run --project pg_protoexport -- batchexport docs/examples/captures docs/examples/exports
dotnet run --project pg_protoexport -- batchexport docs/examples/captures docs/examples/exports --port 5432 --recursive

# Print version (from Nerdbank.GitVersioning) + runtime info. `--version` / `-v` work too.
dotnet run --project pg_protoexport -- version

# Interactive guided tour: walks through every command (starting with ascii) and can run each
# against the bundled sample capture. Non-interactive / piped / --no-run prints the tour only.
dotnet run --project pg_protoexport -- demo
dotnet run --project pg_protoexport -- demo --no-run

# Live capture (writes a .pcapng from a live NIC; needs Npcap/libpcap)
dotnet run --project pg_protoexport -- capture <output.pcapng>
dotnet run --project pg_protoexport -- capture <output.pcapng> --host localhost --port 5432 --duration 30s
dotnet run --project pg_protoexport -- capture <output.pcapng> --quiet         # suppress per-packet console echo
dotnet run --project pg_protoexport -- capture --list-devices

# Pagila sample: capture + run pagila workload in one command
# Writes one .pcapng per scenario, including scenario 00 (startup handshake).
# SslMode defaults to Prefer, so scenario 00 begins with an SSLRequest probe that a
# TLS-off server rejects ('N') and continues in plaintext — a reproducible probe path
# (no Kerberos). Server MUST have TLS off or Prefer upgrades and the capture is encrypted.
# The committed docs/examples/captures were made with PGPORT=5434 PGUSER=postgres PGDATABASE=pagila.
dotnet run --project pg_protoexport.samples.pagila -- capture-and-generate pagila.pcapng
```

## Architecture

The solution has eleven projects plus two samples:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xfischer/pg_protoexport](https://github.com/xfischer/pg_protoexport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
