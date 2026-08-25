---
trigger: always_on
description: > Authoritative project instructions for Claude (and any LLM assistant) working in a codebase that has adopted the **vibecodex** standard. These rules OVERRIDE any default behavior. Follow them exactly. When in doubt, prefer the rule that produces the smaller, more isolated change.
---

# CLAUDE.md — vibecodex

> Authoritative project instructions for Claude (and any LLM assistant) working in a codebase that has adopted the **vibecodex** standard. These rules OVERRIDE any default behavior. Follow them exactly. When in doubt, prefer the rule that produces the smaller, more isolated change.

## Project Purpose

This is a **production FastAPI service**. The codebase follows 18 architectural principles (8 decomposition + 10 integration) that keep the project maintainable as it grows past 30 endpoints, multiple external providers, and a long lifetime. Your job as the AI partner is to add features and fix bugs **without violating any of those principles** — even when the user asks you to "just add it real quick."

If a quick fix would violate a rule, you say so and propose the right shape. You never silently take the shortcut.

---

## Architecture (MANDATORY)

The codebase has exactly four layers. Imports flow **only downward**:

```
Router  → Service → Repository (via Protocol) → ORM/HTTP/S3
                ↘ Provider (via ACL) → external API
```

| Layer            | Lives in                       | What it does                                         | What it CANNOT import                                  |
| ---------------- | ------------------------------ | ---------------------------------------------------- | ------------------------------------------------------ |
| **Router**       | `app/routers/`                 | HTTP only: parse request, call service, format reply | `sqlalchemy`, `httpx`, business logic                  |
| **Service**      | `app/services/`                | Business rules, domain orchestration                 | `sqlalchemy`, `httpx`, `boto3`, FastAPI `Request`      |
| **Repository**   | `app/repositories/`            | Data access (SQL, S3, Redis)                         | other services, FastAPI                                |
| **Provider**     | `app/providers/<vendor>/`     | External API adapter, returns ACL types              | services, repositories, FastAPI                        |
| **Schema**       | `app/schemas/`                 | Pydantic request/response models                     | services, repositories                                  |
| **Models**       | `app/models/`                  | SQLAlchemy ORM                                       | services, providers                                    |
| **Core**         | `app/core/`                    | Config, deps, logging, http clients                  | services, routers (avoid cycles)                       |

**Dependency Inversion via `typing.Protocol`** — services never depend on a concrete repository. They accept an interface:

```python
# repositories/protocols.py
from typing import Protocol
from decimal import Decimal
from app.domain.wallet import Wallet, LedgerEntry

class WalletRepoProtocol(Protocol):
    async def get(self, user_id: str) -> Wallet: ...
    async def lock(self, user_id: str): ...  # async ctx mgr, FOR UPDATE
    async def persist(self, wallet: Wallet, entry: LedgerEntry) -> None: ...
```

```python
# services/wallet/user.py
class WalletUserService:
    def __init__(self, repo: WalletRepoProtocol):
        self._repo = repo
```

```python
# core/deps.py
def get_wallet_service(db: Session = Depends(get_db)) -> WalletUserService:
    return WalletUserService(repo=SQLAlchemyWalletRepo(db))
```

```python
# routers/wallet.py
@router.get("/wallet/me", response_model=WalletResponse)
async def get_wallet(
    user_id: str = Depends(get_current_user_id),
    svc: WalletUserService = Depends(get_wallet_service),
) -> WalletResponse:
    return WalletResponse.from_domain(await svc.get_for(user_id))
```

---

## The 18 Principles as Actionable Rules

### Part A — Safe Decomposition

**Rule A1 — Folder, not file, when a domain splits by type.**
If `routers/X.py` is acquiring multiple disjoint sub-domains (e.g. image / video / audio), convert to package `routers/X/{a.py, b.py, c.py, __init__.py}`. `__init__.py` re-exports a combined `router`. `main.py` doesn't change.

**Rule A2 — Static data lives in `app/data/` (or a `registry.py` next to its domain).**
Pricing tables, model registries, prompt templates, country lists. Never inline a 60-line dict at the top of a service file.

**Rule A3 — Auth and schemas don't live in the router file.**
Auth deps go to `core/<domain>_auth.py`. Pydantic schemas go to `schemas/<domain>/*.py`. Routers stay thin.

**Rule A4 — One file per provider format.**
A vendor that exposes `image` and `video` APIs becomes `providers/<vendor>/{image.py, video.py, _client.py}`. Shared HTTP plumbing lives in `_client.py`.

**Rule A5 — Worker handlers are NOT in the router.**
HTTP enqueue/list/cancel stays in `routers/tasks.py`. Background processing logic lives in `services/task_handlers/<format>.py` and is invoked by your queue worker, not by FastAPI.

**Rule A6 — User-API ≠ Admin-API in the same service file.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yerdaulet-damir/vibe-coding-rules](https://github.com/yerdaulet-damir/vibe-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
