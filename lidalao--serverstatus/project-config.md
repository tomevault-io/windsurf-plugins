---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A management/notification add-on for cppla's [ServerStatus](https://github.com/cppla/ServerStatus) probe. It does **not** modify upstream ServerStatus server code; it wraps the upstream Docker image with node CRUD tooling, a Telegram up/down notifier, a vendored modern web frontend, and systemd-based agent installation. CLI text/prompts are in Chinese ("jj"/"MJJ" are slang for VPS servers); the **web UI is English-only**.

## Layout

```
sss.sh                 # 服务端入口: 安装面板 + 交互式节点管理(纯 bash + jq, 无 python 依赖)
docker-compose.yml     # 编排(project name=sss): srv + web + bot -> 容器 sss-srv-1 / sss-web-1 / sss-bot-1
service/
  bot/{bot.py,Dockerfile}            # Telegram 上下线通知 bot(build)
  web/{index.html,css/app.css,js/app.js,favicon.svg,Dockerfile}  # 自建前端(照搬 tz.huilang.me 配色 + monitor.seaya.link 系统字体栈), build 进 nginx 镜像
agent/{client-linux.py,sss-agent.sh,sss-agent.service}  # 被监控机器侧
```

## Architecture

Three roles communicate through the upstream ServerStatus server, run unmodified as the `cppla/serverstatus:latest` Docker image:

- **Server side (panel host)** — `docker-compose.yml` (project `name: sss`) runs three services, auto-named `sss-srv-1` / `sss-web-1` / `sss-bot-1` (no `container_name`): **`srv`** (upstream `cppla/serverstatus`, TCP report port `35601`) receives agent uploads and writes `json/stats.json`; **`web`** (nginx, frontend baked into the image via `service/web/Dockerfile`) serves the dashboard on `8081` and reads the shared `./json` mounted read-only at the `json/` subpath; **`bot`** (built from `service/bot/`) sends Telegram notifications. `config.json` (`{"servers":[...]}`) is bind-mounted into `srv` and is the single source of truth for which nodes exist. `srv` and `web` share the host `./json` directory — srv writes, web reads.
- **Node management** — implemented **directly in `sss.sh`** (bash + `jq`); there is no `_sss.py` anymore. The menu (`menu_loop`) does view/add/remove/update against `config.json` via `jq` (atomic temp-file + `mv` writes, `sort_by(.name)`), then `docker-compose restart` to reload. Each added node gets a random `username` (`/proc/sys/kernel/random/uuid`) + `password` (`/dev/urandom`, ≥1 digit/lower/upper); `print_agent_cmd` prints the exact `agent/sss-agent.sh` install command.
- **Agent side (monitored machines)** — `agent/client-linux.py` (upstream cppla collector, only modification: `tupd()` stubbed to return zeros) connects to the panel's `35601` and authenticates with the node's USER/PASSWORD, streaming metrics. Installed as a systemd service via `agent/sss-agent.sh`.

`service/bot/bot.py` polls `http://srv/json/stats.json` (the `srv` service hostname on the compose network — auto-named container `sss-srv-1`) every 3s and sends Telegram messages on state changes. **Debounce:** a node must report the same state for 10 consecutive polls (`counterOn`/`counterOff`) before a message fires — suppresses flapping. State is in-memory only.

### Web frontend (`service/web`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lidalao/ServerStatus](https://github.com/lidalao/ServerStatus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
