---
trigger: always_on
description: `feishu-cli` 是一个功能完整的飞书开放平台命令行工具，**核心功能是 Markdown ↔ 飞书文档双向转换**，支持文档操作、消息发送、权限管理、审批查询、知识库操作、文件管理、评论管理等功能。
---

# CLAUDE.md - 飞书 CLI 项目指南

## 项目概述

`feishu-cli` 是一个功能完整的飞书开放平台命令行工具，**核心功能是 Markdown ↔ 飞书文档双向转换**，支持文档操作、消息发送、权限管理、审批查询、知识库操作、文件管理、评论管理等功能。

## 技术栈

| 组件 | 选型 | 说明 |
|------|------|------|
| CLI 框架 | github.com/spf13/cobra | 子命令、自动补全 |
| 飞书 SDK | github.com/larksuite/oapi-sdk-go/v3 | 官方 SDK |
| 配置管理 | github.com/spf13/viper | YAML/环境变量 |
| Markdown | github.com/yuin/goldmark | GFM 扩展支持 |

> 项目结构：`cmd/`（CLI 命令）、`internal/auth`（OAuth）、`internal/client`（API 封装）、`internal/converter`（Markdown 转换器）、`skills/`（Claude Code 技能）。按需 `ls` 查看详情。

## 开发指南

### 构建与测试

```bash
go build -o feishu-cli .          # 快速构建
make build                        # 构建到 bin/feishu-cli
make build-all                    # 多平台构建（发版用，自动注入版本号）
go test ./...                     # 运行所有测试
go vet ./...                      # 静态检查
```

### 质量标准（任何改动提交前必须满足）

1. **实物验证铁律：任何改动，都必须使用实际编译的二进制产物进行验证。**
   先 `go build -o feishu-cli .`（或 `make build`）编译当前代码，再用编译出的二进制
   实跑受影响的命令确认行为符合预期（读操作真实调用；写操作优先 `--dry-run`，必要时
   在测试文档/画板上真实执行）。禁止只凭 `go vet`、单元测试或静态读代码就宣告完成；
   文档/技能中对 CLI 行为的描述，同样以新编译二进制的实跑结果为准。
2. **基线全绿**：`gofmt -l cmd internal` 无输出、`go test ./...`、`go vet ./...` 通过。
3. **结论附证据**：报告"已完成/已修复"必须附带验证命令及其输出（或退出码/截图），
   不做未验证的声明。
4. **可执行文档必须实跑**：README / skills 里给出的命令、脚本、示例，改动后逐条实跑；
   涉及可视化配色的改动，从仓库根运行
   `node skills/feishu-cli-visual/references/workflows/dataviz/scripts/validate_palette.js` 复验定稿色板、
   `node skills/feishu-cli-visual/references/workflows/dataviz/scripts/check_docs.js` 核查文档一致性，全绿才算完成。
5. **Skill 结构校验**：修改 `skills/`、CLI 命令或源码中的 Skill 路径后运行 `make check-skills`；
   该目标会先从当前源码重新构建 `bin/feishu-cli`，再检查 Skill 结构、引用和命令唯一归属。
6. **隐私扫描**：提交前按下方"开发规范"第 6 条检查敏感信息。

发版有更严格的完整清单，见「发布 Release 规范」。

### 开发规范

1. **错误处理**：使用中文错误信息，提供解决建议
2. **命令帮助**：所有命令使用简体中文描述
3. **代码注释**：关键逻辑使用中文注释
4. **提交信息**：遵循 Conventional Commits 规范
5. **指针解引用**：使用 `internal/client/helpers.go` 中的 `StringVal/BoolVal/IntVal` 等工具函数
6. **错误码分支判定**：用 `client.HasAPICode(err, 码)`（词边界安全，`internal/client/api_code.go`），禁止对 err.Error() 做数字 substring 匹配（会撞 log_id 同数字串）
7. **命令组守卫**：`cmd/command_guard.go` 在 Execute 时给所有命令组注入未知子命令守卫（报错+拼写建议+exit 1）；新增命令组无需额外处理，但**不要**给纯分组命令手写 RunE
8. **发送者名字**：读消息统一带 `with_sender_name=true`，服务端回填名经 `internal/client/sender_names.go` 进程级注册表采集，`ResolveSenderNames` 三步解析（服务端回填 → mentions → contact 兜底）
9. **隐私安全（开源项目，必须遵守）**：
   - 代码、文档、技能文件中**禁止出现任何真实的个人邮箱、密码、Token、密钥**
   - 示例邮箱统一使用 `user@example.com`，示例 Token 使用 `cli_xxx`、`u-xxx` 等占位符
   - 新增或修改文件前，检查是否包含 `@bytedance.com`、`@lark.com` 等内部邮箱域名
   - URL 域名使用通用的 `feishu.cn`，**禁止带企业前缀**（如 `bytedance.feishu.cn`）
   - `.env`、`config.yaml` 等含敏感信息的文件已在 `.gitignore` 中排除，禁止提交

### 配置方式

**优先级**：环境变量 > 配置文件 > 默认值

```bash
# 环境变量（推荐）
export FEISHU_APP_ID=cli_xxx
export FEISHU_APP_SECRET=xxx

# 配置文件 (~/.feishu-cli/config.yaml)，通过 feishu-cli config init 初始化
```

## 核心功能

### OAuth 认证（Device Flow）

通过 **OAuth 2.0 Device Flow（RFC 8628）** 获取 User Access Token，用于搜索、审批任务查询等需要用户授权的功能。**无需配置重定向 URL 白名单**（v1.18+ 已删除 Authorization Code Flow）。

**Token 使用策略**（按命令分四类，对应 `cmd/utils.go` 四个 helper）：
- **读类 · User 优先 + Tenant 兜底**（`resolveOptionalUserTokenWithFallback`，约 85 个命令）：`msg history/list/get/mget/thread-messages/resource-download`、`chat list`、`doc read`、`sheet table-get`、`task get/list/subtask list/comment list/tasklist get/list/tasks`、`calendar get/list/primary/agenda/freebusy/suggestion/room-find/event get/list/search/attendee list`、`file meta/stats/list/version list/get/download`、`board image/nodes/export-code/lint`、`user read`、`wiki get/nodes/spaces/export/member list`、`drive pull/push/status`、`vc bot meeting-events`（端点拒收 Tenant Token，User 优先）、**sheet 全家桶**（所有 sheet 子命令含写）等。优先级链：`--user-access-token` → `FEISHU_USER_ACCESS_TOKEN` → `~/.feishu-cli/token.json`（过期自动刷新）→ `config.yaml` 的 `user_access_token` → App Token 兜底。
- **写类 · 默认 Bot 身份**（`resolveOptionalUserToken`）：所有 `add/create/update/delete/move/copy/import/upload/send/reply/forward/merge-forward` 类命令、`comment reply`、`doc content-update / table 写`、`file version revert`、`wiki move-to-drive`、`msg delete`（Bot 自撤回）等。**不会自动加载 token.json**，仅当显式传 `--user-access-token` 或 `FEISHU_USER_ACCESS_TOKEN` 时切到 User Token。`vc bot meeting-join/leave` 同属默认 Bot 身份，但用更严格的 `resolveFlagUserToken`：**只认 `--user-access-token` flag，连 `FEISHU_USER_ACCESS_TOKEN` 环境变量都不读**。
- **必须 User Token**（`resolveRequiredUserToken` / `requireUserToken`）：`search docs/messages/apps`、`approval task query/approve/reject/transfer`、`approval instance get/cancel/cc`、`task my`（`my_tasks`）、`msg pin/reaction/search-chats/flag`、`chat get/update/delete/member`、`vc search/notes/recording/detail`、`vc note detail/transcript`、`task search`、`minutes/mail` 全部（含 `minutes search/apply-permission`、`mail message-modify/message-trash/draft-send`）、`drive secure-label`、`drive upload/download/export/import/move/add-comment/task-result/search`、`calendar rsvp`、`markdown create/fetch/overwrite/diff` 等。失败直接报错。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [riba2534/feishu-cli](https://github.com/riba2534/feishu-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
