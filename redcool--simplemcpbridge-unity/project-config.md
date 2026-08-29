---
trigger: always_on
description: **PowerUtilities 包是外部依赖，不是本仓库的一部分。**
---

# AGENTS.md — SimpleMCPBridge

## PowerUtilities 约束

**PowerUtilities 包是外部依赖，不是本仓库的一部分。**  
如果要修改 PowerUtilities 包的代码，必须先告知用户详情并取得同意后才能改。

## Architecture

Two separate repos that work together:

| Component | Repo | Tech |
|-----------|------|------|
| MCP Server | `SimpleMcpServer` (独立 clone) | Node.js 22+, TypeScript, ws |
| Unity Bridge | 本仓库 (`SimpleMCPBridge`) | C#, Unity 2022.3 |

Data flow: Agent (stdio) → MCP Server → WebSocket → Unity Bridge → Unity API.

## Key Commands

**Server (run from `H:\ai_works\SimpleMcpServer/`):**
- `start.bat` — build + start
- `start-quick.bat` — skip build, start (if dist/ is up to date)
- `node tests\test-e2e.cjs` — E2E test (spawns server, waits for bridge, calls tools)
- `npm run dev` — watch mode via tsx (no build needed)
- `setup.bat` — first-time setup (node check + npm install + build)

**Unity Bridge:**
- 拖 `Prefabs/MCPBridge.prefab` 进场景（或给任意 GameObject 添加 `MCPBridge` 组件）
- 选中后 Inspector 内联显示状态/Bridge ID/错误面板 + IP/Port 字段 + Connect 按钮
- `[ExecuteAlways]` 三态可用（Edit/Play/Player）；isAutoReconnect 断线自动重连
  （含进出 Play Mode、域重载、场景切换）

## Port Conflicts (Common)

The `ws` library's WebSocketServer sometimes leaves the port bound after the
process exits. Before restarting, kill stale processes:

```powershell
Get-Process -Name "node" | Stop-Process -Force
```

If Unity has multiple windows open, more than one bridge might connect —
server keeps only the most recent one.

## Config

- **Server:** `SimpleMcpServer/config.json` — `{ "ip": "127.0.0.1", "port": 45678 }`
- **Bridge (Editor):** 项目 `Assets/SimpleMCPBridge-config/bridge-config.json` — 首次自动从包内
  `Resources/bridge-config.json` 拷贝生成（不存在时创建，已存在则不覆盖）；用户可改，重启生效
- **Bridge (Player):** `Application.persistentDataPath/bridge-config.json` — 同上逻辑
- 兜底: 包内 `Resources/bridge-config.json` 内嵌默认值
- Both use the same format. Cloud deployment: server `ip: "0.0.0.0"`.
- `scene.call_component_method` 权限字段（可选，需重启生效）：
  - `methodBlocklist`: 数组，追加拦截项（`"MethodName"` 或 `"TypeName.MethodName"`，大小写不敏感）；代码默认 6 项
    （`destroy`/`destroyimmediate`/`destroyobject`/`quit`/`quitimmediate`/`disconnect`）**始终生效，无法通过配置移除**
  - `methodAllowlist`: 空数组 = 关闭；非空 = 白名单模式，只放行命中的方法（同上两种格式）；白名单**不会覆盖**黑名单
    —— 调用权限合并语义：代码默认黑名单 + 配置追加黑名单 双重拦截始终优先，白名单只是最后一道放行门槛

## Logs

- **Bridge debug:** `Logs/mcp_bridge_debug.log` (relative to Unity project root)
- **Server stderr:** `SimpleMcpServer/server.err`
- **Unity Editor log:** `$env:LOCALAPPDATA\Unity\Editor\Editor.log`

## editor.eval 安全说明

`editor.eval` 通过 Mono.CSharp 动态编译并在内存中执行任意 C# 代码 —— 等同于完全的 Unity/机器控制（读写任意文件、删除资产、网络访问、启动进程等）。这是 MCP 工具集最强的"逃生舱":当某个场景操作没有专用工具覆盖时,AI 可用 eval 即时补救。

**默认 ON,以用户方便为先**。开发调试、快速原型、补救缺口工具时即时可用,不必先翻配置。若你的环境不可信（共享机器/公网暴露的服务器),关闭它。

**双重 gate（任一关闭即不可用）**:
- **Server 侧 `config.json` → `evalEnabled`**（默认 `true`）:`false` 时 `tools/list` 不暴露 `editor.eval` 给 agent,agent 看不到也就调不到。
- **Bridge 侧 `EditorPrefs SimpleMCPBridge_EvalEnabled`**（默认 `true`）:执行前再检查一次;可用 MCPBridge Inspector 的 toggle 切换。

**风险面**:任何能调用 `/rpc` 的 AI 都能执行任意代码。本工具**不做代码内容过滤**（任意代码无法穷举拦截,黑名单无意义）。安全靠网络层 gate —— `allowedIps` 白名单默认仅本机（`127.0.0.1`/`::1`）。云部署（`ip:0.0.0.0`）前务必扩白名单到可信 IP 段,或直接 `evalEnabled:false`。

**与 `scene.call_component_method` 权限的区别**:后者有方法名黑/白名单（可枚举拦截);eval 是任意代码,只能靠网络层 gate,无方法级过滤。

**关闭方法**（任一即可）:
- `SimpleMcpServer/config.json` 设 `"evalEnabled": false`（重启 server 生效,对所有 agent 隐藏）
- Unity Editor: MCPBridge Inspector 的 eval toggle（立即生效,单机）
- 代码:`EditorPrefs.SetBool("SimpleMCPBridge_EvalEnabled", false)`

## Tools (127)

| Tool | What it does | Platform |
|------|-------------|----------|
| `scene.get_hierarchy` | Scene tree (root→children, with components + positions) | All |
| `scene.get_objects` | Filtered list by nameContains | All |
| `scene.get_objects_by_type` | Objects with a component type | All |
| `scene.get_objects_by_tag` | Objects with a tag | All |
| `scene.get_objects_by_path` | Object at Transform path | All |
| `scene.create_object` | New GameObject with options | All |
| `scene.delete_object` | Destroy by instanceId or path | All |
| `scene.duplicate_object` | Duplicate a GameObject | All |
| `scene.rename` | Rename a GameObject | All |
| `scene.set_active` | Enable/disable a GameObject | All |
| `scene.set_transform` | Set position/rotation/scale | All |
| `scene.set_parent` | Set parent (omit parentId or 0 to unparent to root) | All |
| `scene.set_component_property` | Set field/property on a component | All |
| `scene.set_material` ⚠ | Set material color/texture on Renderer | All |
| `scene.get_components` | List all components on a GameObject | All |
| `scene.get_component_properties` | Get all serializable properties + current values | All |
| `scene.add_component` | Add component by type name | All |
| `scene.remove_component` | Remove a component from a GameObject | All |
| `scene.instantiate_prefab` | Instantiate a prefab from project Assets (PrefabUtility.InstantiatePrefab — 保留 prefab 关联,实例改动传导 prefab;材质资产化依赖此关联推断同级目录) | Editor |
| `scene.save_current` | Save current scene | Editor |
| `scene.load_scene` | Load a scene (Editor: open asset; Play/built: SceneManager, single/additive) — ⚠ after load ALL instanceIds go stale, re-fetch hierarchy | All |
| `scene.save_prefab` | Save a GameObject (instanceId/path) as a prefab asset (overwrites existing) | Editor |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redcool/SimpleMCPBridge_Unity](https://github.com/redcool/SimpleMCPBridge_Unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
