---
trigger: always_on
description: 你是一名资深前端开发工程师，正在开发一款叫 Prodivix 的工业级浏览器端可视化前端开发工具。以下是这款工具的核心架构。
---

# Prodivix Agents 开发指南

你是一名资深前端开发工程师，正在开发一款叫 Prodivix 的工业级浏览器端可视化前端开发工具。以下是这款工具的核心架构。

## 当前全局阶段

- 当前产品位置：`G1 Passed / G2 Foundation`（G0、G1 `ProductGateStatus=Passed`，G2 `ProductGateStatus=In Progress`）。
- `specs/roadmap/global-phases.md` 是 Global Phase 的唯一来源，`specs/roadmap/g0-closure-evidence.md` 保存 G0 的可重复验证边界。
- G1 已形成 revision-bound TypeScript/JavaScript/CSS/SCSS/GLSL/WGSL Language Capability、独立 WebGL2/WebGPU Shader Compile Capability、跨编辑器 CodeSlot、external adapter 与 orphan lifecycle、跨领域 code refactor、PIR-current ↔ canonical React/JSX + standalone CSS controlled round-trip、DTCG Token/Resolver、Asset Semantic Provider、完整 Blueprint/Component/Collection 产品表面、唯一 durable 生产写入链，以及独立 React/Vite install/typecheck/test/build/browser-smoke Gate。G2 已建立 transport-neutral ExecutionProvider/ExecutionJob、instance-owned Execution Session coordinator、provider-neutral Executable Project Snapshot、共享 Browser Project Runtime Host、相互独立的 Preview/Test Provider、NodeGraph/Animation same-context provider，以及 Remote codec/client/provider projection、Remote Preview/Build Bundle/Test Report result、授权 artifact resolver、有界 HTTP envelope/content transport、Backend user-auth gateway/durable execution grant、Control Plane Core、PostgreSQL adapter/integration Gate、独立 HTTP service、Worker Agent、rootless Podman sandbox/GitHub Isolation Gate、D2 durable event/log、content-addressed artifact blob、总预算与 retention、短期 capability Remote Preview Host，以及同一 Golden neutral snapshot 的 Browser Preview/Test 与 Remote Preview/Test/Build contract matrix/GitHub Gate；真实 Golden snapshot 的 rootless Preview/Test/Build、internal install network + hostname/443 allowlist proxy、install/runtime 断网硬切、transport-neutral sanitized `network.request` 与 Execution Center Network 视图也已进入该 Gate，Blueprint Run Mode 已可显式选择 Browser/Remote provider。DataSourceDocument/DataOperationReference current contract、`data-source` typed Workspace document、Data Semantic Contribution、PIR/Collection durable binding 与显式 lifecycle mapping、Data invocation/mock-live adapter registry/lifecycle execute kernel、typed trigger/input mapping 与 deterministic dispatch coordinator、PIR-current v1.6 query activation/input 和 mutation event durable authoring、Inspector/Semantic/Workspace Transaction/`data-input-transform` CodeSlot、exact document JSON Schema preflight、deterministic retry/pagination、SHA-256 partitioned bounded cache policy kernel、owner/version fenced optimistic CRUD executor、独立 HTTP adapter、session-scoped deterministic mock fixture/stateful CRUD namespace、content-addressed Executable Snapshot v4 fixture provisioning/Remote codec、Browser/Remote provider runtime asset projection，以及 React/Vite generated document/route/input-change query activation、semantic typed input、Blueprint mock mutation CRUD/query revalidation和 public client live HTTP/schema/retry/pagination/cache/optimistic runtime 已建立；Executable Snapshot 现在显式投影 mock/live runtime manifest，Browser iframe 的 sanitized Network trace 通过 exact-origin/message-source bridge 关联 active Job，Remote finite Preview 的 server/edge Data trace 通过 exact active-job、generation-fenced、bounded Session observation 进入同一产品视图。Browser fetch composition、transport-neutral environment snapshot/permission/material ports、短期 resolution lease、zone/execution-class/isolation/field permission matrix、Backend production Environment/Secret store、principal/session partition、durable grant/audit、execution-bound Remote live gateway、mutation replay fence 与 capability-origin CSP 也已落地。React/Vite Data runtime target manifest 现在默认 `static-client`；server/edge Data 只有显式 execution parent gateway target 才能通过 compile Gate，并将 `network`/`environment-binding` 要求传播到 snapshot、Remote provider 与 request，Browser/ZIP export fail closed；Workspace Test 使用强制 `mock-only` target，live manifest 不能降级穿透。Remote mutation 现已支持显式 `invocation-key` policy、HTTP adapter idempotency header、attempt-invariant opaque key 与 v3 next-attempt ledger；无 contract 仍固定 attempt 1。Remote 当前 durable 输出已通过 runtime-core guard、Worker 出站与 Control Plane 入站双 Gate 覆盖 request/snapshot/cache/log/diagnostic/trace/artifact/test-report/crash Secret canary，命中只保留安全 `EXE-5004` 与固定终态原因。Structured Console 现在统一投影 state/log/diagnostic/artifact/trace/application observation，通过 exact frame Session fence、条数/字节预算和 generated/bridge/core/copy 多层 credential redaction 进入 Execution Center；manual recovery 等待旧 Job terminal 后创建新 request，保留旧事件且不自动重放 mutation。Remote Preview Terminal 已通过独立 strict wire、Backend owner gateway、短期 token rotation、Control Plane ephemeral broker、worker mailbox/lease fence、rootless inner PTY、cursor reconnect、跨 chunk canary 与 Execution Center 产品路径落地；Browser 显式 unsupported。Runtime FS strict diff artifact、PTY-close capture handshake、授权 resolver、Execution Center Files 与 revision/lifecycle-fenced whole-file CodeArtifact add/modify/delete VFS 原子采纳已落地。继续聚焦完整 Remote reconnect/artifact/quota/worker-loss recovery、KMS/key rotation、SourceTrace 调试旅程与第二 framework target。
- Canonical Workspace VFS 是作者态唯一真相。PIR、NodeGraph、Animation、Data Source、Code、Design Token、Design Token Resolver、Assets、Config 与 RouteManifest 是 Workspace 内由各领域 owner 管理的文档或清单；PIR 不是整个项目的单一巨型 JSON。

```mermaid
flowchart TD
    %% 核心作者态文件系统
    subgraph VFS [Workspace VFS]
        direction TB

        WorkspaceCore[workspace.json / route-manifest.json]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prodivix/prodivix](https://github.com/prodivix/prodivix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
