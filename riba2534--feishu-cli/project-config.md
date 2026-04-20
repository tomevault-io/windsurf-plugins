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

### 开发规范

1. **错误处理**：使用中文错误信息，提供解决建议
2. **命令帮助**：所有命令使用简体中文描述
3. **代码注释**：关键逻辑使用中文注释
4. **提交信息**：遵循 Conventional Commits 规范
5. **指针解引用**：使用 `internal/client/helpers.go` 中的 `StringVal/BoolVal/IntVal` 等工具函数
6. **隐私安全（开源项目，必须遵守）**：
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

**Token 使用策略**：
- **默认使用 App Token**（租户身份）：wiki、msg、calendar、task 等 55+ 个命令默认通过 `resolveOptionalUserToken` 使用 App Token，不会自动加载 User Token
- **显式使用 User Token**：通过 `--user-access-token` 参数或 `FEISHU_USER_ACCESS_TOKEN` 环境变量覆盖为用户身份
- **必须 User Token**：搜索命令（`search docs/messages/apps`）通过 `resolveRequiredUserToken` 走优先级链：
  1. `--user-access-token` 参数
  2. `FEISHU_USER_ACCESS_TOKEN` 环境变量
  3. `~/.feishu-cli/token.json`（过期自动用 refresh_token 刷新）
  4. `config.yaml` 中的 `user_access_token`
- **审批任务查询**：`approval task query` 会调用 `/authen/v1/user_info` 推断 `open_id`，缓存到 `~/.feishu-cli/user_profile.json`，`auth logout` 时自动清理

**登录命令四种模式**：
- `auth login --scope "..."`：显式请求 scope，阻塞轮询
- `auth login --domain <name> --recommend`：按业务域申请推荐 scope（推荐）
- `auth login --json`：JSON 事件流输出（AI Agent 推荐，配合 `run_in_background`）
- `auth login --no-wait --json` + `auth login --device-code <code> --json`：两步模式

**scope 策略**：登录时显式声明 scope，不再依赖后台全量兜底。`offline_access` 和 `auth:user.id:read` 自动注入。AI Agent 执行业务前应先 `auth check --scope "REQ_SCOPES"` 预检。

**审批输出模式**：不传 `--output` 输出文本摘要；`--output json` CLI 归一化 JSON；`--output raw-json` 原始响应。

### Markdown ↔ 飞书文档双向转换

**导入**：`feishu-cli doc import doc.md --title "文档" --verbose`
**导出**：`feishu-cli doc export <doc_id> -o output.md`

支持的语法：标题、段落、列表（无限深度嵌套）、任务列表、代码块、引用（QuoteContainer）、Callout（6 种类型）、表格、分割线、图片（默认 `--upload-images` 上传）、链接、公式、粗体/斜体/删除线/下划线/行内代码/高亮

### Mermaid / PlantUML 图表转画板

**推荐 Mermaid**，导入时自动转画板。支持 8 种 Mermaid 类型：flowchart（含 subgraph）、sequenceDiagram、classDiagram、stateDiagram-v2、erDiagram、gantt、pie、mindmap。PlantUML 支持时序图、活动图、类图、用例图、组件图、ER 图、思维导图等全部类型。

### 表格智能处理

- **自动拆分**：飞书 API 限制单表格最多 9 行 × 9 列，超出自动拆分并保留表头/首列
- **列宽自动计算**：中文 14px，英文 8px，最小 80px，最大 400px
- **单元格多块支持**：单元格内可混合 bullet/heading/text

### 图表导入容错

- 服务端错误自动重试（最多 10 次，1s 间隔）
- Parse error / Invalid request parameter 不重试，直接降级为代码块
- 失败回退：删除空画板块，在原位置插入代码块

### 三阶段并发管道（导入架构）

1. **阶段一（顺序）**：按文档顺序创建所有块，收集图表和表格任务
2. **阶段二（并发）**：图表 worker 池 + 表格 worker 池并发处理
3. **阶段三（逆序）**：处理失败图表，降级为代码块

**CLI flags**：`--diagram-workers`（默认 5）、`--table-workers`（默认 3）、`--diagram-retries`（默认 10）、`--upload-images`（默认开启）、`--image-workers`（默认 2，API 限制 5 QPS）

## 命令速查

完整命令清单见 [README.md](README.md) 和对应 skill 文档。关键入口：

```bash
# 认证
feishu-cli auth login --domain <name> --recommend       # 按业务域登录
feishu-cli auth check --scope "REQ_SCOPES"              # 预检 scope
feishu-cli auth status                                   # 查看授权状态

# 文档导入/导出（核心功能）
feishu-cli doc import input.md --title "..." --upload-images --verbose
feishu-cli doc export <doc_id> -o output.md
feishu-cli doc content-update <doc_id> --mode <mode> --markdown "..."
#   mode: append / overwrite / replace_range / delete_range / insert_after

# 多维表格（统一用 --base-token，底层 base/v3 API）
feishu-cli bitable {create|get|copy} ...
feishu-cli bitable {table|field|record|view|role} <action> ...
feishu-cli bitable view view-{filter|sort|group|visible-fields|timebar|card}-{get|set} ...
feishu-cli bitable advperm {enable|disable} --base-token ...
feishu-cli bitable {data-query|workflow list} ...

# 邮件（User Token 必需，默认存草稿，--confirm-send 才发送）
feishu-cli mail {triage|send|draft-create|reply|forward|message|thread} ...

# 云盘增强
feishu-cli drive {upload|download|export|import|move|add-comment|task-result} ...

# 视频会议与妙记（全部需 User Token）
feishu-cli vc {search|notes|recording} ...
feishu-cli minutes {get|download} --minute-tokens ...

# 其他模块：doc / msg / sheet / calendar / task / tasklist / chat / wiki / file / perm / board / search / user / dept / comment / media
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [riba2534/feishu-cli](https://github.com/riba2534/feishu-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
