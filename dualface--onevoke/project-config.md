---
trigger: always_on
description: 本文件是 Onevoke 仓库自身的开发规则. 仓库对外发布的工作流规则在 `rules/`, 那些文件是交付物, 不是本仓库的开发指引.
---

# Repository Guidelines

本文件是 Onevoke 仓库自身的开发规则. 仓库对外发布的工作流规则在 `rules/`, 那些文件是交付物, 不是本仓库的开发指引.

## 本仓库特例

- 本仓库第二阶段安全角色 `CSA` 和 `Hacker` 一律标记 N/A, 不运行; `PM` 和 `QA` 保持适用.
- 审核 base 以来全部改动都是 Markdown 规则或文档时, 不运行审核. 只要包含任一脚本, 代码或其他非 Markdown 文件, 就按适用规则运行 `PM` 和 `QA`; `CSA` 和 `Hacker` 仍按上一条标记 N/A.
- 对外发布的分支模型固定为 `main` 稳定分支加 `develop` 集成分支, 不提供其他长期分支或集成分支选项; 缺少 `develop` 时从 `main` 自动初始化.
- 功能或修复改完后默认: 在 `develop` 提交 → fast-forward 合入 `main` → 推送 `develop` 与 `main` → POSIX 运行 `./install.sh`, 原生 Windows 运行 `install.ps1` 更新本机安装; 用户另有指示时除外.

## Project Structure & Module Organization

- `rules/ONEVOKE-AGENTS.md` 是发布规则的入口, 只放分册索引, 优先级和默认行为. 其余分册由它的分册表按需引用: `BASE-RULES.md` 跨项目通用条款, `KANBAN-RULES.md` 看板行为契约, `GIT-RULES.md` Git 工作流, `REVIEW-RULES.md` 审核契约, `CODE-RULES.md` 架构与代码质量契约. 它们是面向用户和 Agent 的对外接口, 改动前确认与 `bin/` 下实现一致. 全部装到 `~/.agents/` 下的同名文件.
- `install.sh` 与 `install.ps1` 分别是 POSIX 和原生 Windows 安装器. 两者遍历 `bin/*` 和 `rules/*.md`, 把全部普通文件直接覆盖到 `~/.local/bin/` 与 `~/.agents/`, 包括 `ONEVOKE-AGENTS.md`; Windows 安装器不修改用户 `PATH`, 必须提示用户把 `~/.local/bin` 加入 `PATH`, 命令通过 `.cmd` 包装入口运行. Windows 安装器读取既有配置语言时必须实际执行候选 Python 3, `py -3` 失败后继续探测 `python.exe`, 再探测 `python3.exe`; 跳过 0 字节的 Windows App Execution Alias (会打开微软商店的重定向器), 已安装的商店版 Python 从其 AppX 包目录取真实 `python.exe`; 不得从 PowerShell 当前 FileSystem provider 位置或 Win32 进程当前目录选择同名程序, 拒绝这些候选后须继续探测 PATH 中后续同名程序. 若存在 `share/kanban-web/`, 同步安装到 `~/.local/share/onevoke/kanban-web/` 供 `kanban web` 使用. 升级时检测已退役的 `codex-review.sh`、`claude-review.sh` 和 `grok-review.sh`, 提示用户且仅在明确确认后删除; 拒绝或无输入时保留. `~/.agents/AGENTS.md` 不存在时, POSIX 创建指向 `ONEVOKE-AGENTS.md` 的相对符号链接, Windows 优先创建硬链接并回落到符号链接, 两者都不得用独立副本冒充入口; 已有任何同名入口时保持不变. 唯一稳定 stdout 按 locale 为 `Onevoke 已安装` 或 `Onevoke installed`; 全局安装最后必须用绝对路径运行 `onevoke welcome`. POSIX `install.sh --project <目录>` 把同一套载荷只装到目标 Git 项目主 worktree 的 `.onevoke/` (命令、规则、share), 幂等写入本地 `/.onevoke/` exclude, 目标从任一 worktree 指定都归一到主 worktree; 不创建、修改或探测 HOME 下 Onevoke 路径, 不运行 welcome, 不修改 PATH, 不迁移或卸载全局安装. 非 Git、无效参数、目录或符号链接目标拒绝且不回落全局安装. 项目安装成功时 stdout 在稳定安装行之后给出项目本地 `onevoke` 与 `kanban` 绝对路径. 同名目标是目录或 Windows reparse point 时须在写任何文件前拒绝, 防止安装器把源文件写入错误边界.
- `bin/onevoke_config.py` 是 `onevoke` 与 `kanban` 共用的配置边界, 配置默认在 `~/.config/onevoke/config.json`, 测试用 `ONEVOKE_CONFIG` 隔离. `install_paths()` 按当前入口解析作用域: 入口位于 `.onevoke/bin/` 时为项目模式, 路径落在 Git 主 worktree 的 `.onevoke/` (config, rules, bin, share), 否则为全局模式并保持 `~/.config/onevoke/config.json`, `~/.agents`, `~/.local/bin`, `~/.local/share/onevoke`; 源码树的 `bin/` 与 `rules/` 不得判为项目安装. `ONEVOKE_CONFIG` 仍覆盖 `config_path()`. `project_install_paths(project)` 供安装器把目标归一到主 worktree; `ensure_project_git_exclude(project)` 幂等写入 `/.onevoke/` 到本地 `info/exclude`, 保持既有权限, 复用 `onevoke_fs` 的 no-follow 追加与锁, 不安全链接边界必须失败. 配置写入必须校验 schema; POSIX 用同目录临时文件加 `os.replace()` 原子替换, 权限为 `0600`. Windows 的 `configured_language`, 读取和写入必须从卷/UNC anchor 逐分量 no-follow, 拒绝符号链接、junction 等 reparse point; load 在不共享 WRITE/DELETE 的同一固定句柄上完成读取、schema 校验和旧配置 DACL 迁移, 无效配置不迁移 ACL; save 仅收紧本次新建的配置目录, 临时文件必须先变为当前用户独占的受保护 DACL 再写入, 最后相对固定父句柄原子替换; 不得收紧既有祖先目录. 任一权限或安全后端失败必须报错. `language` 为 `cn`/`en`, 默认 `cn`, 可在 welcome 设置; 生效优先级为 `--lang` (经 `ONEVOKE_LANG_CLI` 跨进程传递) > 配置 > 环境变量. `launcher` 允许 `auto`/`tmux`/`tmux-session`/`herdr`/`foreground`/`console`, POSIX 默认 `auto`, Windows 默认 `console`; `console` 仅支持 Windows; `auto` 与 `herdr` 仅支持 POSIX. `kanban_agent` 是默认执行 Agent, 可选的 `kanban_agents` 段按 `large`/`small` 两种任务卡规模分别指定执行 Agent, 缺失的规模回落到 `kanban_agent`, 未知规模或非法 Agent 拒绝, `kanban_agent_for(config, kind)` 与 `execution_agents_in_use(config)` 是 `kanban`/`onevoke` 共用的读取口; `models` 段保存 kanban 与 review 的模型和推理档位, 缺失层级用默认值补齐, 未知键拒绝; `model`/`large_model`/`small_model` 允许空串表示用 CLI 默认模型. `review_stages` 为 `PM`/`CSA`/`Hacker`/`QA` 各指定 `auto`/`skip`/`required`, 缺省全为 `auto`. 它同时是脚本, `review-model <agent>` 子命令输出两行 (`<model>` 与 `<effort>`, model 可为空行) 供 `onevoke_review.py` 读取; `review-stages` 按角色顺序输出四行环节策略; `configured-language` 在配置文件存在时输出 `cn`/`en`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dualface/onevoke](https://github.com/dualface/onevoke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
