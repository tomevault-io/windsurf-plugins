---
trigger: always_on
description: `cdxgen` is a universal, polyglot CLI tool that generates valid CycloneDX Bill-of-Materials (BOM) documents in JSON format. It produces SBOM, HBOM, CBOM, OBOM, SaaSBOM, VDR, and CDXA outputs for source code, containers, VMs, live operating systems, and supported hardware hosts. Supports CycloneDX spec versions `1.4`–`1.7` (default: `1.7`, with HBOM currently targeting `1.7`). cdxgen features a best-in-class, native **JSON Signature Format (JSF)** implementation for BOM signing, providing robust 
---

# Skill: OWASP cdxgen (CycloneDX BOM Generator)

## Description

`cdxgen` is a universal, polyglot CLI tool that generates valid CycloneDX Bill-of-Materials (BOM) documents in JSON format. It produces SBOM, HBOM, CBOM, OBOM, SaaSBOM, VDR, and CDXA outputs for source code, containers, VMs, live operating systems, and supported hardware hosts. Supports CycloneDX spec versions `1.4`–`1.7` (default: `1.7`, with HBOM currently targeting `1.7`). cdxgen features a best-in-class, native **JSON Signature Format (JSF)** implementation for BOM signing, providing robust authenticity and non-repudiation capabilities. Unlike basic signing tools, our implementation fully supports granular signatures (signing individual components, services, and annotations), parallel Multi-Signatures (`signers`), and sequential Signature Chains (`chain`). When the optional companion binaries from `@cdxgen/cdxgen-plugins-bin` are available, cdxgen also enriches container/rootfs and live-OS scans with Trivy/osquery-powered metadata, Linux GTFOBins runtime context, platform trust posture, and Go Evinse evidence through the `golem` helper. HBOM collection is dynamically provided by the optional `@cdxgen/cdx-hbom` library on supported `darwin/arm64`, `linux/amd64`, and `linux/arm64` hosts. CBOM mode can also extract cryptographic algorithm inventory from JavaScript and TypeScript source through lightweight AST analysis.

## ✅ When to Invoke

- User requests an SBOM/BOM for a repository, directory, container image, or live OS.
- User requests a hardware BOM / HBOM for the current host.
- User needs dependency inventory, license resolution, or vulnerability triage context.
- User wants to export to Dependency-Track, sign/validate a BOM, convert CycloneDX JSON to SPDX JSON-LD, or generate evidence/callstacks.
- User wants a predictive audit of an existing CycloneDX BOM with `cdx-audit`, especially for npm or PyPI package compromise posture.
- User wants Go dependency evidence, usage scopes, call graph context, data-flow/crypto-flow evidence, local replacement review, vendoring/license evidence, crypto components, or security-sensitive API signals from `evinse -l go` with Golem.

## 📦 Prerequisites & Installation

| Requirement   | Detail                                                                                                   |
| ------------- | -------------------------------------------------------------------------------------------------------- |
| **Runtime**   | Node.js ≥ 20 (≥ 22.21 recommended for native proxy support)                                              |
| **Java**      | ≥ 21 required for C/C++/Python/CBOM analysis. Fails silently or produces incomplete BOMs with Java 8/11. |
| **Install**   | `npm i -g @cyclonedx/cdxgen` or `pnpm dlx @cyclonedx/cdxgen`                                             |
| **Container** | `docker run --rm -v $(pwd):/app:rw -t ghcr.io/cyclonedx/cdxgen:master /app`                              |

Notes:

- The optional `@cdxgen/cdxgen-plugins-bin` packages provide native helpers such as Trivy and osquery.
- The same plugin package also provides `golem` for Go Evinse semantic evidence when a platform binary is available.
- Container and `rootfs` scans can surface repository source components plus trusted-key cryptographic assets when those binaries are present.
- Container and `rootfs` scans also emit `cdx:container:unpackagedExecutableCount` and `cdx:container:unpackagedSharedLibraryCount` metadata properties so agents can spot native file inventory that was not traced to OS package ownership.
- Linux live-OS profiles include hardening-oriented `sysctl_hardening` and `mount_hardening` snapshots plus GTFOBins enrichment on privileged and network-active runtime rows.
- The optional `trustinspector` helper adds macOS code-signing/notarization and Windows Authenticode/WDAC properties across large host inventories without truncating path inspection after the first few hundred paths.
- macOS live-OS OBOM collection uses the bundled osquery binary in shell mode and may still require Full Disk Access or elevated privileges for some tables.

## 💻 Core Syntax

```bash
cdxgen [path] [options]
```

- `path` defaults to `.` (current directory)
- All boolean flags accept `--no-` prefix to invert behavior
- Config precedence: `CLI args` > `CDXGEN_* env vars` > `.cdxgenrc`/`.cdxgen.json`/`.cdxgen.yml`/`.cdxgen.yaml`

## 🔑 Key Parameters & Profiles

| Category       | Flag                      | Purpose                                                                                                                                                          |
| -------------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdxgen/cdxgen](https://github.com/cdxgen/cdxgen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
