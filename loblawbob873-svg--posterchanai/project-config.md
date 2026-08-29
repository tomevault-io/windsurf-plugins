---
trigger: always_on
description: validates against the catalogue (`/meme/effect`, `/meme/apply-effect`, `/effects/run`) aliases
---

# CLAUDE.md

Guidance for working in this repo. Keep changes small, clean, and consistent with the
patterns below.

## What this is

PosterChanAI — a self-hosted FastAPI app: streaming LLM chat (OpenAI-compatible `/v1/`),
image gen, TTS/STT, email/news/torrents, a file manager, a **Nostr client + relay**, and
**Telegram + fediverse bots**. Single-admin, multi-user, **Postgres**-backed.

## Run / dev

- Entry point: `python run.py` (uvicorn, **single worker**, port from `POSTERCHANAI_PORT`,
  default **3051**). On this deployment the Intel Arc box runs `posterchanai.service`
  (port 3051); `nas.lan` runs `posterchanai`. **There is no separate image service** — the
  port-3052 `posterchanai-xpu-image.service` was retired when the venvs were unified, and 3052 is
  now the RELAY (`run.py --role relay`). `systemctl is-active` answers `inactive` for a unit that
  was never installed, which reads exactly like "stopped and should be running"; `systemctl cat`
  (`No files found`) is the question worth asking.
- venv at **`venv-unified/`** (`venv-unified/bin/python`) — there is no `venv/` on this deployment.
  Quick checks: `venv-unified/bin/python -m py_compile <files>`, and `-m pyflakes` for undefined
  names (what `sync.sh`'s pre-push gate runs).
- Logs: `journalctl -u posterchanai.service` (the fediverse `[PROXY] CONNECT ... SOCKS5`
  errors are unrelated federation noise — ignore when debugging features).

## Test — `./test.sh`, before the deploy and after it

One command runs everything: `pytest tests/`, `pytest tests/client/`, and all 20+ browser-driven
`scripts/check_*.py` (mobile layout, Meme Builder, the windowed desktop, Notes/Calendar/Contacts/
Mail/vault/Web Search/Files, the composer and quote modals, the terminal, the extension). ~10 min.
`--live URL` adds the checks that need a running instance; `--docker` runs the lot in a container
that **publishes no ports**, so it is safe on a node already serving 3051. `--brief` prints a fixed
report between markers for the node agent to paste verbatim — the format is rendered in Python, for
the same reason `/logs` is: a small model gathers reliably and retells badly.

**The list of checks is DISCOVERED, not typed** — a new `scripts/check_*.py` joins the suite the
moment it is written, and one that is unregistered runs anyway and says so. **Exit 2 means "could
not run"** and is reported as a SKIP with its reason, never as a pass. Two rules for a new check,
both because they run concurrently: read the chrome port from `PC_CHECK_PORT` and the profile from
`PC_CHECK_PROFILE` (four scripts used to share 9473), and never write into the working tree's live
state — a test that touched `streamserver/mediamtx.pid` passed on a laptop and PermissionError'd on
every node that was actually serving. See `docs/TESTING.md`.

## Deploy — always via `sync.sh`

`./sync.sh` does `git commit -a -m fix && git push`, then restarts local services and
resets/restarts `nas.lan`. **`git commit -a` does NOT stage new untracked files** — `git add`
any new file before running it, or it ships a broken (ImportError) tree to every node.
`sync.sh` deploys **code only**, not Python deps (use `install.sh` option 6 for deps).

**Two remotes — `origin` is the NOSTR repo (production), `github` is the public mirror. Gitea is
gone.** `origin` is `nostr://npub1fdtthaq…/relay.poster.place/posterchanai`, served by the built-in
GRASP host at `https://poster.place/git/<owner-npub>/posterchanai.git` (see
`docs/GIT_OVER_NOSTR.md`). That is what `sync.sh`/`git push` deploys to **production**, so push there
**first**. The `github` remote (`github.com/loblawbob873-svg/posterchanai`) is a **public mirror**
whose default branch is `main`, mapped from local `master`: push to it explicitly with
`git push github master:main`. **Both remotes get every push, with no prompting** — finish a change
by committing and pushing to `origin` first (deploy), then mirroring the same commits to `github`,
so the public mirror never falls behind production. Keep local `master` tracking `origin` (so plain
`git push` deploys, not publishes).

```
git push                     # or ./sync.sh — origin/production first
git push github master:main  # mirror, same commits, every time
```

**A deploy pulls EVERY node, even when it restarts nothing.** The pull is free; only the restart
costs an outage, and `scripts/deploy_targets.py` decides that separately. Skipping `sync.sh` for a
UI-only change and hand-pulling router.lan left **nas.lan 3 commits behind**, running old code with
nothing in any log to say so. `sync.sh` now pulls both nodes, waits on the GPU **only** if something
is actually restarting, and ends by verifying local/origin/github/nas/router are all on the commit —
exiting **1** on drift rather than reporting a green deploy. Guarded by
`tests/test_sync_deploy_flow.py`.

Push authorization is a **Nostr signature, not a connection**: only a maintainer of
`30617:<owner>:posterchanai` can move a ref, and the `pre-receive` hook reads the **hosting node's**
(nas) relay Postgres. server1 and nas run separate relays with separate event stores, so the repo
announcement lists **`wss://poster.place/git`** — that endpoint proxies to *nas's* relay, which is why

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loblawbob873-svg/posterchanai](https://github.com/loblawbob873-svg/posterchanai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
