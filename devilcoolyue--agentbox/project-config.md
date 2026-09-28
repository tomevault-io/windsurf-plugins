---
trigger: always_on
description: 给代码助手/维护者的项目说明。用户向的部署与功能文档见 `README.md`（英文）和 `README_CN.md`（中文）；这里记录实际代码结构、开发命令和容易踩坑的约定。
---

# AGENTS.md

给代码助手/维护者的项目说明。用户向的部署与功能文档见 `README.md`（英文）和 `README_CN.md`（中文）；这里记录实际代码结构、开发命令和容易踩坑的约定。

## 项目是什么

`agentbox` 是一个 Go 单二进制服务端：在 Linux 服务器上通过 Docker 为每个浏览器会话拉起一个容器，容器里运行 Claude Code / Codex CLI。浏览器通过 HTTP/WebSocket 使用对话、终端、文件、共享目录、账号池和内网反向隧道。

两个入口：

- `cmd/agentbox`：服务端。加载 `config.json`，对 `data_dir` 加 flock，初始化 store/docker/server 后监听 HTTP。
- `cmd/abox-link`：用户本机反向隧道客户端。无参数时开本机控制台 `127.0.0.1:7801`；带 `--server` 时走命令行无头模式。

运行环境：服务端依赖 Docker daemon；生产部署目标是 Linux + systemd。macOS 上可以 `go build`/`go test`，但不能完整验证容器链路。

## 仓库地图

| 路径 | 作用 |
|---|---|
| `cmd/agentbox/main.go` | 服务端入口与信号；`internal/app` 管理数据锁、启动与依赖清理。 |
| `cmd/abox-link/main.go` | 隧道客户端入口；面板模式与 `--server` 无头模式分流。 |
| `internal/app` | 启动编排、数据目录独占锁与依赖收尾。 |
| `internal/workspace` | 会话创建/启停/删除、模板与凭证播种、活动引用与空闲回收。 |
| `internal/credentials` | 账号凭证读取/保存、轮换同步、续期与账号级可取消锁。 |
| `internal/config` | 配置 schema、校验、运行时修改与原子写回。所有设置变更必须经 `Config.mutate`/`ApplySettings`/账号方法。 |
| `internal/server` | HTTP API、鉴权、用户/账号/设置、会话、文件、聊天 WS、终端 WS、隧道、账号出口代理（`proxy*.go`）、监控、空闲回收、凭证同步、Git 变更审查（`git.go`）、用量计量与额度（`usage.go`/`quota.go`/`usagelog.go`）。 |
| `internal/usage` | 回合用量归一化、定价快照、结算编排及 Claude/Codex 终端扫描。 |
| `internal/store` | SQLite(`data/state.db`)：sessions/users/tokens/usage_events/quotas/credit_ledger；首次打开会导入旧版 `state.json`。 |
| `internal/backup` | 版本化 tar.gz 备份、SQLite 在线快照、清单/哈希验证、恢复到新目录；CLI 在 cmd/agentbox/backup.go。 |
| `internal/safefs` | 基于 os.Root 的受限目录句柄、普通文件读取、原子写入及跨目录不覆盖重命名；详见 docs/architecture/filesystem-boundaries.md。 |
| `internal/gitx` | 网页 Git 的容器执行策略：argv、环境隔离、超时与窄执行接口；禁止宿主机 Git 降级。 |
| `internal/dockerx` | Docker Engine API 封装：容器生命周期、exec PTY/stream、stats、镜像/挂载检查。 |
| `internal/agent` | Claude/Codex 适配层：headless 命令、标题生成、凭证播种、Claude HUD、Codex app-server 协议。 |
| `internal/archivex` | 上传压缩包解压（防 zip-slip/符号链接/解压炸弹）与工作区 zip 下载。 |
| `internal/tunnel` | yamux 隧道协议、白名单、端口映射。 |
| `internal/linkapp` | abox-link 客户端实现：配置、面板、守护/自启、重连监督器；`static/` 是面板前端，随 `cmd/abox-link` 独立 `go:embed`。 |
| `internal/web` | 嵌入前端静态资源（`static/js` 是 TS 编译产物）；`AGENTBOX_WEB_DIR` 可改为磁盘热加载。 |
| `web/src` | 主控制台前端 TypeScript 源码，`npm run build` 编译到 `internal/web/static/js`。 |
| `images/agent` | 会话容器镜像 Dockerfile；内置 Claude Code、Codex CLI、tmux、claude-hud。 |
| `scripts` | 镜像构建/自动升级、abox-link 交叉编译、域名与账号登录辅助脚本。 |
| `deploy` | systemd 单元（服务、镜像更新、数据备份）、logrotate、安装/发布脚本、生产参数模板。 |

## 常用命令

```bash
# 基础校验
go build ./...
go test ./...
npm run check          # 前端类型检查（tsc --noEmit）

# 前端：改了 web/src/*.ts 必须重新构建，产物要一起提交
npm ci                 # 首次或依赖变动时
npm run build          # web/src/*.ts -> internal/web/static/js/*.js

# 构建两个二进制
go build -o agentbox ./cmd/agentbox
go build -o abox-link ./cmd/abox-link

# 构建会话容器镜像
./scripts/build-image.sh

# 交叉编译 abox-link 到 data/abox-link/ 供 Web UI 下载
# 注意：客户端依赖较新服务端接口，通常先 deploy 服务端再跑这个
./scripts/build-clients.sh

# 检查 npm 上 Claude Code / Codex 新版并重建镜像
./scripts/auto-update-image.sh

# 本地试跑（先 cp config.example.json config.json 并改 auth_token/accounts）
./agentbox -config config.json

# 前端热改：不嵌入，直接吃磁盘文件（配合 npm run watch 自动重新编译 TS）
AGENTBOX_WEB_DIR=internal/web/static ./agentbox -config config.json
npm run watch          # 另开一个终端；改完 .ts 刷新浏览器即可

# 生产部署/日常发布（Linux，需要 root）
sudo ./deploy/install.sh
sudo ./deploy/deploy.sh
```

特殊测试：

```bash
# 默认跳过；真实调用本机 codex app-server，消耗少量额度
CODEX_LIVE_TEST=1 go test -run TestRunCodexTurnLive ./internal/agent/
```

## 生产环境发布

生产机的具体地址、目录、域名等敏感信息不写进版本库，集中放在
`deploy/production.env`（已 gitignore，模板见 `deploy/production.env.example`）。
执行下面任何命令前先加载它：

```bash
source deploy/production.env   # 提供 PROD_SSH / PROD_DIR / PROD_LISTEN / PROD_DOMAIN / PROD_URL
```

| 变量 | 含义 |
|---|---|
| `PROD_SSH` | 生产机 SSH 目标（已配置免密登录，自动化时加 `-o BatchMode=yes`） |
| `PROD_DIR` | 生产机上的仓库目录 |
| `PROD_LISTEN` | 服务监听地址（如 `127.0.0.1:8180`） |
| `PROD_URL` | 公网访问地址 |

systemd 服务名固定为 `agentbox.service`。

发布前用 `systemctl show agentbox -p WorkingDirectory -p ExecStart` 核对实际布局。下文 `deploy.sh` 仅适用于服务直接运行在源码仓库内的旧布局；若启动路径是 `/opt/agentbox/current/agentbox`，使用独立发布目录与 `deploy/release.py` 的暂存、激活流程（见 `docs/architecture/deployment-layout.md`），不要为了适配源码目录重写现有 systemd 单元。

只有用户明确要求部署到生产时才执行以下操作。发布会重启服务并短暂断开现有
HTTP/WebSocket 连接，不是滚动发布。

### 发布前检查

本地先确认改动范围并跑完整校验：

```bash
git status --short --branch
go build ./...
go test ./...
```

然后确认远端服务与仓库状态。远端可能有用户留下的未跟踪文件；保留它们，不要
使用 `git clean`、`git reset --hard` 或 `rsync --delete`：

```bash
ssh -o BatchMode=yes "$PROD_SSH" 'systemctl is-active agentbox'
ssh -o BatchMode=yes "$PROD_SSH" "git -C $PROD_DIR status --short --branch"
```

### 已提交代码发布

代码已提交并推送到 `origin/main` 时，在生产仓库仅快进拉取，再运行部署脚本：

```bash
ssh -o BatchMode=yes "$PROD_SSH" "git -C $PROD_DIR pull --ff-only origin main"
ssh -o BatchMode=yes "$PROD_SSH" "$PROD_DIR/deploy/deploy.sh"
```

如果远端有已跟踪文件改动，先判断来源；不要擅自覆盖或回退。冲突时停止发布并向
用户说明。

### 本地未提交改动发布

用户要求直接预览尚未提交的本地改动时，只用 `rsync -R` 定向同步本次改动涉及的
源码文件，保持仓库相对路径。不要同步整个仓库，也不要覆盖 `config.json`、
`accounts/`、`data/` 或远端未跟踪文件。例如：

```bash
rsync -azR \
  internal/web/static/index.html \
  internal/web/static/css/shell.css \
  internal/web/static/js/theme.js \
  "$PROD_SSH:$PROD_DIR/"

# 注意：同步的是 npm run build 的产物 internal/web/static/js/*.js，
# 不是 web/src/*.ts —— 生产机没有 node，不会自己编译。先在本地构建好。

ssh -o BatchMode=yes "$PROD_SSH" "git -C $PROD_DIR diff --check"
ssh -o BatchMode=yes "$PROD_SSH" "$PROD_DIR/deploy/deploy.sh"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devilcoolyue/agentbox](https://github.com/devilcoolyue/agentbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
