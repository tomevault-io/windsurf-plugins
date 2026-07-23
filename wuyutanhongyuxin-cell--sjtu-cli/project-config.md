---
trigger: always_on
description: > 本文件是 SJTU-CLI 项目的 Claude Code 指令，基于"通用 AI 编程规范"填充项目专属区域。
---

# CLAUDE.md — SJTU-CLI 项目规范

> 本文件是 SJTU-CLI 项目的 Claude Code 指令，基于"通用 AI 编程规范"填充项目专属区域。
> Claude Code 每次启动自动读取。

---

## [项目专属区域]

### 项目名称
**SJTU-CLI** —— 上海交通大学"交我办"命令行工具

### 一句话描述
给 SJTU 师生用的终端 CLI，通过 JAccount 扫码登录后，一行命令查课表 / 成绩 / 一卡通 / 通知 / Canvas，输出支持 YAML/JSON，方便 AI Agent 调用。

### 技术栈（Rust 实现）
- **语言**：Rust（stable, edition 2021）
- **包管理 / 构建**：cargo
- **CLI 框架**：clap 4（derive API）
- **HTTP 客户端**：reqwest（features: `cookies` / `json` / `rustls-tls` / `gzip`）
- **异步运行时**：tokio（`rt-multi-thread` + `macros`）
- **浏览器自动化**：headless_chrome（纯 Rust CDP 驱动）
- **Cookie 提取兜底**：rookie（browser-cookie3 的 Rust 等价）
- **HTML 解析**：scraper（html5ever）
- **QR 终端显示**：qrcode + ANSI 色块
- **序列化**：serde + serde_json + serde_yml + toml
- **表格 / 颜色**：comfy-table + owo-colors
- **TTY 检测**：is-terminal
- **时间**：chrono（含 `serde` feature）
- **路径**：directories（跨平台 `~/.sjtu-cli/` 定位）
- **金额**：rust_decimal（一卡通消费金额，禁用 f64）
- **错误**：anyhow（bin 层）+ thiserror（lib 层）
- **日志**：tracing + tracing-subscriber
- **测试**：内置 `#[test]` + tokio-test + mockito
- **Lint**：cargo clippy + cargo fmt
- **CI**：GitHub Actions（stable × windows / ubuntu / macos）

### 项目结构
```
sjtu-cli/
├── Cargo.toml                       # 依赖 + 构建配置
├── Cargo.lock
├── rust-toolchain.toml              # 固定 stable
├── README.md
├── SKILL.md                         # AI Agent 使用指南
├── SCHEMA.md                        # 输出信封契约
├── CLAUDE.md                        # 本文件
├── .env.example
├── config.example.toml
├── src/
│   ├── main.rs                      # 二进制入口，调用 cli::run
│   ├── lib.rs                       # 模块 re-export；VERSION 常量
│   ├── cli.rs                       # clap 命令枚举 + 派发
│   ├── config.rs                    # ~/.sjtu-cli/ 路径管理
│   ├── cookies/                     # Cookie 模型 + 文件 I/O（S2 瑕疵修复时拆分）
│   │   ├── mod.rs                   # Cookie / Session struct + redacted 三元组键
│   │   ├── io.rs                   # load / save / clear（主 + 子 session）
│   │   └── tests.rs                 # redacted + path 防覆盖单测
│   ├── error.rs                     # thiserror 统一异常
│   ├── output.rs                    # Envelope + TTY 检测 + YAML/JSON/Table 切换
│   ├── auth/                        # 登录模块
│   │   ├── mod.rs
│   │   ├── qr_login.rs              # ★ headless_chrome QR 扫码
│   │   ├── qr_render.rs             # 终端 ANSI QR 重绘
│   │   ├── browser_extract.rs       # rookie cookie 兜底
│   │   └── cas/                     # 子系统 CAS 跳转（S2 拆分）
│   │       ├── mod.rs               # cas_login + follow_redirect_chain
│   │       ├── client.rs            # 注入主 session 构造 reqwest Client
│   │       └── tests.rs             # mockito 跳转链单测
│   ├── apps/                        # 每个子系统一个文件
│   │   ├── mod.rs
│   │   ├── jwc.rs                   # 教务（课表/成绩）
│   │   ├── library/                 # 图书馆借阅（weijieyue.lib.sjtu.edu.cn:8080）
│   │   │   ├── mod.rs
│   │   │   ├── client.rs            # OAuth dance + Client + 3 业务方法
│   │   │   ├── http.rs              # cookie 注入 + fetch_json
│   │   │   ├── models.rs            # Loan / HistoryRow / Fine serde struct
│   │   │   ├── throttle.rs          # 300ms 节流
│   │   │   └── tests_parse.rs       # fixture + mockito e2e
│   │   ├── mail/                    # 邮箱（mail.sjtu.edu.cn Zimbra SOAP）
│   │   │   ├── mod.rs               # re-export + MailClient 入口
│   │   │   ├── client.rs            # SSO 跟链 + ZM_AUTH_TOKEN + 4 只读方法
│   │   │   ├── http.rs              # build_client + SOAP POST + token 抽取
│   │   │   ├── soap.rs              # SOAP envelope builder（只读 4 类，红线注入 read=0）
│   │   │   ├── parser.rs            # quick-xml 流式解析 SearchResponse / GetMsgResponse
│   │   │   ├── extract.rs           # ZM_AUTH_TOKEN 从 Set-Cookie / SOAP body 提取
│   │   │   ├── models.rs            # Mail / MailFull / Address struct
│   │   │   ├── throttle.rs          # 300ms 节流
│   │   │   └── tests_parse.rs       # fixture parse 单测 + mockito e2e
│   │   ├── card.rs                  # 一卡通
│   │   ├── notifications.rs         # 交我办通知
│   │   └── canvas.rs                # Canvas LMS
│   ├── commands/                    # CLI 子命令实现
│   │   ├── mod.rs
│   │   ├── auth_cmds.rs             # login/logout/status/whoami
│   │   ├── schedule.rs              # schedule/today/week/next
│   │   ├── grades.rs                # grades/gpa
│   │   ├── card.rs                  # card/card history
│   │   ├── library/                 # 图书馆借阅命令
│   │   │   ├── mod.rs
│   │   │   ├── data.rs
│   │   │   └── handlers.rs
│   │   ├── mail/                    # 邮箱命令
│   │   │   ├── mod.rs
│   │   │   ├── data.rs              # MailListData / MailListRow / MailReadData
│   │   │   └── handlers.rs          # cmd_mail_list / cmd_mail_read
│   │   ├── notifications.rs
│   │   └── canvas.rs
│   └── models/                      # struct（derive Serialize/Deserialize）
│       ├── mod.rs
│       ├── course.rs
│       ├── grade.rs
│       ├── card_record.rs
│       └── notification.rs
├── tests/                           # 集成测试
├── tasks/
│   ├── todo.md                      # 任务清单
│   └── lessons.md                   # 经验积累
└── .github/workflows/ci.yml
```

### 当前阶段

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wuyutanhongyuxin-cell/SJTU_CLI](https://github.com/wuyutanhongyuxin-cell/SJTU_CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
