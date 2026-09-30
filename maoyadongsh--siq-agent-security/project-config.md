---
trigger: always_on
description: siq-agent-security 本地二进制（Go；模块路径仍为 `apps/agentshield`）。上层约束见仓库根 `AGENTS.md`；本文只写本模块的就近规则。实现规格是 `docs/agentshield-dev-spec-v1.md`——**先读它，再改代码；规格与代码冲突时先改规格。** W7 本地台账计划见 `docs/agentshield-local-ledger-dev-plan-v1.md`（新 API / 状态文件须先回写规格）。
---

# apps/agentshield 工作约定

siq-agent-security 本地二进制（Go；模块路径仍为 `apps/agentshield`）。上层约束见仓库根 `AGENTS.md`；本文只写本模块的就近规则。实现规格是 `docs/agentshield-dev-spec-v1.md`——**先读它，再改代码；规格与代码冲突时先改规格。** W7 本地台账计划见 `docs/agentshield-local-ledger-dev-plan-v1.md`（新 API / 状态文件须先回写规格）。

## 模块与职责

| 包 | 职责 | 状态 |
| --- | --- | --- |
| `internal/canon` | 与 CPython `json.dumps(sort_keys=True, separators=(",",":"))` 逐字节一致的规范化 JSON | 完成 |
| `internal/rulepack` | 内嵌规则包、外部包 Ed25519 验签、防降级、fail-closed 回退 | 完成 |
| `internal/threat` | 静态分析器（Python `threat_analysis.py` 移植，AST 层缺席） | 完成 |
| `internal/signing` | 本地 Ed25519 身份、文档/字节签名与验签 | 完成 |
| `internal/inventory` | 只读盘点：平台配置、Skill、Hermes profiles、OpenClaw agents.list、MCP 客户端配置（`mcp_server`）；可选 `--connectors-dir` **exec** 子进程（不 import `connectors/*`） | 规格 §3.5 |
| `internal/export` | 脱敏导出包 `agentshield.export.v1`（无私钥/token/参数原文） | 规格 §3.8.1.3 |
| `internal/controlsync` | `sync --control-api` → Edge `POST /edge/v1/batches`；缺凭据跳过；失败不改本地决策 | 规格 §2.4 |
| `internal/admission` | frontmatter、哈希、限额、决策表、Skill Card | 完成（决策表变更需同步规格 §3.6.4 与 `dispositions.go`）|
| `internal/grant` | declared → allowlist / DesiredPolicy；状态机；`PatchDesired`；读回 effective | 完成（`CompilePolicy` 与 Python `artifact_hash` 对等）|
| `internal/receipt` | 决策引擎、污点/trifecta、哈希链、Verify；block 下无 host 的出网 exec deny；hold 审批后签名预留与不确定恢复 | N06/R02 组件批次 |
| `internal/state` | 状态目录、token、admission/grant/policy/assets/findings/audit 文件态存储 | 完成 |
| `internal/ledger` | 台账投影 + 资产生命周期 refresh（G7）；confirm/dismiss/drift/accept | 完成 |
| `internal/server` | `/v1/*` HTTP（loopback + Host 允许列表 + 决策/管理分权 + 配对）+ 无 secret 的 `/ui-config.json` + embed UI | DEV02-A |
| `internal/adapterinstall` | `adapter install/uninstall/status`：写主机钩子，先备份可还原 | 完成 |
| `internal/openshell` | probe / 网络 `policy set` / 读回；不调 `create_generation` | 完成 |
| `internal/ui` | embed `apps/web` 本地模式构建产物（`npm run build:local`） | 完成 |
| `cmd/agentshield` | 子命令入口（含 `admit`/`grant`/`adapter`/`openshell`/`serve`/`export`/`sync`/`release-manifest`/`manifest-verify`） | 完成 |
| `internal/skillmanifest` | 发布清单构建、Ed25519 验签、诚实 support_matrix | 完成 |
| `internal/skillcontext` | Skill 执行上下文（SEC，`skill-execution-context/v1`）签发/验证/撤销；verified 归属唯一路径；决策时全量重验实例、会话 binding、grant digest、安装与目标内容 | N05/R01 组件批次 |

## 硬性规则

1. **仅标准库。** 需要第三方依赖（如 tree-sitter）先立 ADR。交叉编译 `linux/{amd64,arm64}`、`darwin/arm64`、`windows/amd64` 必须始终通过；不得引入 `fcntl`/`syscall` 平台专属调用，文件/锁独占用 `O_EXCL`。不可变版本允许按 ADR-012 用标准库 `os.Link` 将已写完的同目录暂存文件排他发布；文件系统不支持时显式失败，不回退覆盖或暴露半写文件。
2. **规则包是共享文件。** `internal/rulepack/data/threat_rules.v1.json` 必须与 `apps/control-api/app/data/threat_rules.v1.json` 逐字节一致（`TestEmbeddedPackMatchesControlPlaneCopy` 锁定）。改模式：RE2 可编译、CPython 语义等价、附边界用例、两侧测试都跑。
3. **对等优先于"更好"。** `threat` 的输出（sha256 / rule_id / line / excerpt_sha256 / excerpt）必须与 Python 相同；想改行为先在 Python 侧改并同步语料，再移植。
4. **签名只签规范化字节。** 所有文档签名 = `Ed25519(canon.Marshal(doc 去掉 signature))`，十六进制 128 位；回执链签 `hash` 字符串字节。任何新文档类型都走 `signing`，不得自建序列化。
5. **状态目录之外不写。** 路径由 `state` 包解析（`SIQ_AGENT_SECURITY_STATE_DIR` 覆盖，兼容旧名 `AGENTSHIELD_STATE_DIR`）；目录 0700、文件 0600；只追加或新建，禁止原地改写准入/签发/回执文件。 本用户个人体验开发周期按 ADR-039/ADR-044/ADR-047 增加限定例外：经明确确认并复验的安装、移除或更新操作可写对应 Skill 目标及同 profile 私有操作目录，采用排他发布、权限失效与归属校验恢复；不得执行候选内容或覆盖/删除未知用户对象。
6. **不执行被分析内容。** 不 `import` Skill、不解压嵌套压缩包、不跟随符号链接；git 来源用 `--depth 1` 并禁用 hooks。可选 `--connectors-dir` 仅 **exec** connector 二进制（规格 §3.5），禁止 import `connectors/*`。
7. **模型不是权威。** 任何未来的 LLM 语义层只能产生 `inferred` 事实或 `info` finding，不能改 verdict / action / status。
8. **fail-closed 表是合同。** `block` 模式下服务不可达、超时、401、非法响应 = 拒绝；普通 policy 的 `audit_only`/`warn` 保留 allow + `advisory_action`。按当前用户开发目标（ADR-0015），无效 Authority 在所有模式下 hard deny，不得被 advisory 放宽。每个适配器必须有对应负向测试。
9. **日志只记类别。** 拒绝原因、异常消息不得包含规则内容、参数、文件内容或密钥。
10. **批准不等于执行。** hold 通过后必须先追加唯一 `hold_reservation` 才能允许外部工具；重复/并发预留拒绝。已预留且无 observation 时只能报告 `uncertain`，不得因超时或权限撤销掩盖可能发生的副作用。管理员人工结案只追加 `hold_reconciliation`，不能再执行旧预留或宣称 exactly-once。

## 测试要求

Windows WorkBuddy 安装目标按 `docs/windows-workbuddy-skill-install-v1.md` 的限定增量执行：用户级仅写已确认配置根下的 `skills`，项目级仅写已登记且已确认项目下的 `.codebuddy/skills`；各自私密事务目录位于对应已验证根的 `.siq-agent-security-installs`，不进入原生 Skill 扫描根。父目录创建事实只追加并签名，操作前复验目标身份，保留未知用户对象；此例外不授予普通项目文件写入或运行权限。

Windows 资源事实按 `docs/windows-resource-profile-spec-v1.md` 的限定例外，允许 `internal/runtimepath/*_windows.go` 使用标准库 syscall/unsafe 只读查询盘符映射、文件/目录句柄、卷和目录大小写属性；不更改资源、盘符或 ACL，不提权，不把路径事实本身当作 Authority。测试可在临时目录内建立并清理 junction，不修改用户对象。

Windows 私密状态按 dev-spec §2 的限定例外，允许 `internal/privatefs/*_windows.go` 使用标准库 `syscall`/`unsafe` 查询文件句柄安全描述符、构造受限 DACL 并在新对象创建时传入。禁止改已有对象 ACL、提权、调用外部权限工具或让其他平台直接引用 Windows API；保留兼容屏障和仅标准库约束。

ACL 负向测试允许仅由 `_test.go` 引用的 `internal/acltest` 辅助包使用同类标准库 API 修改本次测试临时根内的合成对象 DACL，并在结束时还原测试夹具；不得用于产品代码或真实用户对象。

Windows Writer 恢复按 dev-spec §2.3 的限定例外，允许在 `_windows.go` 中使用标准库 `syscall`/`unsafe` 调用进程只读查询和文件句柄重命名 API；独占创建仍用 `O_EXCL`，不引入第三方依赖、进程终止、提权或新的平台服务。跨平台文件不得直接引用这些 API，其他平台构建保持通过。

N01 Windows 迁移暂存清理按 `docs/n01-state-protocol-design-20260913.md` 的限定例外，允许同样通过标准库 Win32 句柄删除本次创建且身份复验一致的暂存链接；禁止通过清除只读位删除硬链接，禁止清理未知对象。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maoyadongsh/siq-agent-security](https://github.com/maoyadongsh/siq-agent-security) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
