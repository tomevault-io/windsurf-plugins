---
trigger: always_on
description: 新项目 bootstrap — 一次性配置项目变量 / 写 sync+approve script（远程项目）/ 跑 /init 扫项目结构 / init 3 个 session snapshot / 加 MEMORY.md 索引 / 提醒开 3 个 claude tab (writer/explainer/data)。**只在开新项目时跑一次**，跑完项目装好。后面日常开发（开 phase branch / codex review loop / cherry-pick / squash-rebase merge）按已有 memory 规则自动走，不在本 skill 里。用法 `/feature-flow`。
---


# feature-flow — 新项目 bootstrap

**只在一个新项目刚开始 / 还没装过本 workflow 时跑一次。** 跑完项目变量都填好、helper script 就位（远程项目）、`/init` 扫好项目结构、3 份 session snapshot 初始化好、MEMORY.md 索引齐、3 tab 配置说明也告诉用户了 — 之后用户日常开 phase / commit / merge 直接按 memory 规则走（见 [Bootstrap 完成后的日常开发](#bootstrap-完成后的日常开发)）。

> **不是**每开一个 phase 都跑一次。每开 phase 直接走日常流程即可。

## 已 bootstrap 检测（每次启动 skill 先跑）

skill 一启动**先检测**当前项目是否已 bootstrap：

```bash
PROJECT_MEMORY_DIR=<推断>
PROJECT_ROOT=<推断>
DEV_HOST=<已知或问>

ok=true
for f in project_writer_handoff.md project_data_handoff.md project_explainer_handoff.md; do
  test -f "$PROJECT_MEMORY_DIR/$f" || ok=false
done
for key in project_writer_handoff project_data_handoff project_explainer_handoff feedback_three_claude_roles; do
  grep -q "$key" "$PROJECT_MEMORY_DIR/MEMORY.md" || ok=false
done
test -f "$PROJECT_ROOT/CLAUDE.md" || ok=false
# 远程项目还要求 helper 存在；本地项目不要求 helper
if [ "$DEV_HOST" != "local" ]; then
  ls "$PROJECT_ROOT"/sync-*-review.sh >/dev/null 2>&1 || ok=false
  ls "$PROJECT_ROOT"/approve-*-diff.sh >/dev/null 2>&1 || ok=false
fi
```

如果 `ok=true` → 项目已 bootstrap。**回一句**"项目已 bootstrap 过，没事可做。日常开发请按下方 §Bootstrap 完成后的日常开发 那张表走"并退出。**不要**重做 step。

---

## Bootstrap 步骤（按顺序跑）

### Step 1 — 收集项目变量

下面这些值要在 bootstrap 时全部确定。能从环境推断的优先推断，剩下问用户。**注意 `PROJECT_NAME` / `REPO_NAME` / `REPO_SLUG` 是三个独立 var**（一个项目可能 workspace 叫 `MyResearch`，里面 repo 是 `CoolModel`，路径里又要小写 `coolmodel`）。

| 变量 | 例 (illustrative) | 推断方式 |
|---|---|---|
| `${PROJECT_ROOT}` | `~/Documents/your-project` | `pwd`（或问用户） |
| `${PROJECT_NAME}` | `your-project` | local workspace name；`basename ${PROJECT_ROOT}` |
| `${DEV_HOST}` | `your-dev-host` 或字面值 `local` | 询问；如本地开发填 `local` |
| `${DEV_REPO}` | `/path/on/remote/your-repo` | 询问（`DEV_HOST=local` 时与 `${PROJECT_ROOT}` 可能不同 — 多 repo 的情况） |
| `${REPO_NAME}` | `your-repo` | repo 真名；`basename ${DEV_REPO}` |
| `${REPO_SLUG}` | `your-repo` | lowercase shell-safe slug；`echo "$REPO_NAME" \| tr 'A-Z' 'a-z'` |
| `${FORK_REMOTE}` | `origin` 或 `myfork` | `git remote -v` 拿候选名让用户选 |
| `${TARGET_BRANCH}` | `main` 或 `dev` | 询问主集成 branch；典型 `main` / `master` / `dev` / `<proj>_dev` |
| `${GIT_NAME}` / `${GIT_EMAIL}` | `your-name` / `you@example.com` | 读 memory `reference_git_identity*`；没有问用户一次然后存进 memory |
| `${PROJECT_MEMORY_DIR}` | `~/.claude/projects/<your-project-slug>/memory` | 按 cwd 推断 claude project 目录 |
| `${HANDOFFS_DIR}` | `~/.claude/projects/<your-project-slug>/handoffs` | 同上；bootstrap 不用，但日常 codex review 必用，存这里方便后续 |

> 用 `AskUserQuestion` 一次性问完缺的几项，不要每个变量单独问。

### Step 2 — 远程 SSH setup + reachable check（仅 `DEV_HOST != local`）

#### 2a. Reachable check

```bash
ssh -o BatchMode=yes "$DEV_HOST" 'echo OK && hostname && whoami'
```

- 成功（打印 `OK <hostname> <user>`）→ 跳过 2b，进 Step 3
- 失败 → 进 2b 走 walkthrough

> `BatchMode=yes` 强制禁 interactive prompt（密码 / passphrase / host-key 都不弹）→ 没 key 直接 exit non-zero。

#### 2b. SSH setup walkthrough（仅 2a 失败时）

⚠️ **claude 不能替用户输密码**。下面命令必须用户**自己在 prompt 输** `! <command>` 跑（`!` 前缀让 claude code 跳出主对话执行 interactive shell，密码 / passphrase 才能进得去）。

**Step 2b-1: 收集真实远程账户信息**

用 `AskUserQuestion` 问用户两项（如果用户没说，默认 `DEV_HOST` 当 alias name）：

| 问题 | 例 |
|---|---|
| `${REMOTE_USER_AT_HOST}` | `you@host.example.com` (真 user@host, 不是 alias) |
| `${SSH_KEY_PATH}` | `~/.ssh/id_ed25519_${DEV_HOST}` (默认 ed25519 + alias 为后缀) |

**Step 2b-2: 检查 / 生成 ssh key**

```bash
test -f "$SSH_KEY_PATH" && echo "key 已存在" || echo "缺 key，让用户生成"
```

缺 key → 告诉用户在 prompt 跑（**用户自己跑**）：

> ```
> ! ssh-keygen -t ed25519 -f ${SSH_KEY_PATH} -C "${DEV_HOST}-key"
> ```
>
> （keygen 会问 passphrase；不想要每次输 → 直接回车留空。设了 passphrase 后续要 `ssh-add ${SSH_KEY_PATH}` 装进 agent 才免密。）

**Step 2b-3: 推 public key 到远程**

```
! ssh-copy-id -i ${SSH_KEY_PATH}.pub -o StrictHostKeyChecking=accept-new ${REMOTE_USER_AT_HOST}
```

- **这一步 ssh 会问一次远程账户密码** — 用户自己输。输完就免密了。
- `StrictHostKeyChecking=accept-new` 自动 trust 第一次见的 host key（CI 友好 + 用户少一次 prompt）。

**Step 2b-4: 写 `~/.ssh/config`**

`Write` 工具追加 / 创建 `~/.ssh/config`（先 `grep -q "^Host ${DEV_HOST}\b"` 检查是否已有同名 alias；有就跳过，避免重复）：

```sshconfig
Host ${DEV_HOST}
  HostName <REMOTE_USER_AT_HOST 的 host 部分>
  User <REMOTE_USER_AT_HOST 的 user 部分>
  IdentityFile ${SSH_KEY_PATH}
  ServerAliveInterval 60
  ControlMaster auto
  ControlPersist 4h
  ControlPath ~/.ssh/cm-%r@%h:%p.sock
```

> `ControlMaster + ControlPersist` 启用连接复用，后续 ssh / scp 共享同一条 master，第一次握手后 4 小时内的命令都秒级。

**Step 2b-5: 回到 2a 重测**

走 2a `ssh -o BatchMode=yes "$DEV_HOST" 'echo OK'`。还失败 → print 错误明细给用户（typo / 路径 / 防火墙 / 远程 sshd 配置），**停止 bootstrap**。仍成功不下去就别强推 step。

### Step 3 — 写 sync + approve helper script（仅 `DEV_HOST != local`）

> ⚠️ **写文件时必须把所有 `${...}` 替换成本项目的 concrete literal values**（如 `your-dev-host` / `/path/on/remote/your-repo` / `your-repo` / `your-repo` 等）。生成后的脚本**不依赖外部环境变量**，独立可执行。下面是模板，写文件前替换。

写入两个文件并 `chmod +x`：

#### `${PROJECT_ROOT}/sync-${REPO_SLUG}-review.sh`（模板，写入时 substitute concrete values）

```bash
#!/usr/bin/env bash
# Sync remote dev tree to local read-only review snapshot (for codex rescue).
set -euo pipefail


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eacsai/feature-flow](https://github.com/eacsai/feature-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
