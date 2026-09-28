---
trigger: always_on
description: <a href="https://github.com/simonepri/refined-antigravity-acp">
---

<p align="center">
  <a href="https://github.com/simonepri/refined-antigravity-acp">
    <img src="assets/logo.svg" alt="Refined Antigravity ACP Logo" width="320">
  </a>
</p>

<h1 align="center">Refined Antigravity ACP</h1>

<p align="center">
  <!-- Implementation -->
  <a href="https://www.typescriptlang.org/">
    <img src="https://img.shields.io/badge/language-TypeScript-3178C6?logo=typescript&amp;logoColor=white" alt="Written in TypeScript">
  </a>
  <a href="https://nodejs.org/">
    <img src="https://img.shields.io/badge/runtime-Node.js_>=22-339933?logo=node.js&amp;logoColor=white" alt="Node.js 22+">
  </a>
  <a href="https://pnpm.io/">
    <img src="https://img.shields.io/badge/package_manager-pnpm-F69220?logo=pnpm&amp;logoColor=white" alt="pnpm">
  </a>
  <br>
  <!-- Quality & Tooling -->
  <a href="https://oxc.rs/">
    <img src="https://img.shields.io/badge/formatter-oxfmt-orange?logo=rust&amp;logoColor=white" alt="oxfmt">
  </a>
  <a href="https://oxc.rs/">
    <img src="https://img.shields.io/badge/linter-oxlint-orange?logo=rust&amp;logoColor=white" alt="oxlint">
  </a>
  <a href="https://publint.dev/">
    <img src="https://img.shields.io/badge/packaging-publint-2A52BE?logo=npm&amp;logoColor=white" alt="publint">
  </a>
  <a href="https://github.com/trumppet/fallow">
    <img src="https://img.shields.io/badge/dead_code-fallow-5C2D91?logoColor=white" alt="fallow">
  </a>
  <a href="https://github.com/rhysd/actionlint">
    <img src="https://img.shields.io/badge/workflows-actionlint-2088FF?logo=githubactions&amp;logoColor=white" alt="actionlint">
  </a>
  <br>
  <!-- Verification -->
  <a href="https://github.com/simonepri/refined-antigravity-acp/actions/workflows/ci.yml">
    <img src="https://img.shields.io/github/actions/workflow/status/simonepri/refined-antigravity-acp/ci.yml?branch=main&amp;label=CI&amp;logo=githubactions&amp;logoColor=white" alt="CI status">
  </a>
  <a href="https://vitest.dev/">
    <img src="https://img.shields.io/badge/tests-vitest-729B1B?logo=vitest&amp;logoColor=white" alt="Vitest unit tests">
  </a>
  <br>
  <!-- Distribution -->
  <a href="https://www.npmjs.com/package/@simonepri/refined-antigravity-acp">
    <img src="https://img.shields.io/npm/v/@simonepri/refined-antigravity-acp?logo=npm&amp;logoColor=white" alt="npm version">
  </a>
  <a href="https://github.com/simonepri/refined-antigravity-acp/stargazers">
    <img src="https://img.shields.io/github/stars/simonepri/refined-antigravity-acp?style=flat&amp;logo=github&amp;logoColor=white" alt="GitHub stars">
  </a>
  <a href="https://github.com/googleapis/release-please">
    <img src="https://img.shields.io/badge/released_with-Release_Please-4285F4?logo=google&amp;logoColor=white" alt="Released with Release Please">
  </a>
  <a href="https://github.com/getpaseo/paseo">
    <img src="https://img.shields.io/badge/ecosystem-Paseo_ACP_Provider-20744A?logo=buffer&amp;logoColor=white" alt="Paseo ACP Provider">
  </a>
  <a href="license">
    <img src="https://img.shields.io/github/license/simonepri/refined-antigravity-acp" alt="MIT license">
  </a>
</p>

<p align="center">
  <strong>🤦 A Google Antigravity ACP binary that actually works.</strong>
</p>

---

## Overview

**Refined Antigravity ACP** is a proxy wrapper around Google's official Antigravity ACP binary (`agy_acp_server.par`).

Google's binary executes models, agent loops, and tool calls. This proxy intercepts the ACP stream between editor and server to fix upstream crashes and deadlocks, normalize MCP traffic, and integrate with **Paseo**, **Zed**, and other ACP clients.

```mermaid
flowchart LR
    subgraph Editors["Supported ACP Clients"]
        PaseoUI["Paseo<br>(ACP Agent Provider)"]
        ZedUI["Zed Editor<br>(Stdio Agent)"]
        OtherUI["Neovim / JetBrains / Custom<br>(Standard ACP)"]
    end

    subgraph Wrapper["Refined Antigravity ACP (Proxy & Hardening Layer)"]
        direction TB

        Supervisor["Process Supervisor<br>• Subprocess Lifecycle & Health<br>• Transparent Crash Recovery<br>• Multi-Session State Cache"]

        subgraph Pipeline["Bidirectional ACP Pipeline"]
            direction TB
            Outbound["Outbound Stream<br>• Request & Option Normalization<br>• Workspace Context Injection<br>• MCP Port & URL Rewriting"]
            Inbound["Inbound Stream<br>• Output & Stream Sanitization<br>• Progress & Plan Synthesis<br>• Interruption Leak Cleanup"]
            Telemetry["Telemetry & Diagnostics<br>• Real-time Stderr Event Tracking<br>• SQLite Checkpoint & History Repair<br>• Token Usage Extraction"]
        end

        McpProxy["Loopback MCP Proxy Pool<br>• Dynamic Endpoint Remapping<br>• Protocol Version Adaptation"]

        Supervisor <--> Pipeline
        Supervisor <--> McpProxy
    end

    subgraph Upstream["Google Official Backend"]
        Kernel["agy_acp_server.par<br>(Google Subprocess)"]
        LocalDb[(Local SQLite Store<br>Conversations & Steps)]
        GeminiCloud["Google DeepMind / Gemini Cloud"]

        Kernel <-->|gRPC / HTTPS| GeminiCloud
        Kernel <-->|WAL Journal| LocalDb
    end

    Editors <-->|Stdio NDJSON CLI| Supervisor
    Supervisor <-->|Standard ACP NDJSON| Kernel
    Pipeline -.->|Direct Read / Heal| LocalDb
    McpProxy <-->|HTTP Rewriting| Kernel


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simonepri/refined-antigravity-acp](https://github.com/simonepri/refined-antigravity-acp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
