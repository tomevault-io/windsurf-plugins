---
trigger: always_on
description: Guidance for AI agents (and humans) working in the **Shvatka** codebase — an
---

# AGENTS.md

Guidance for AI agents (and humans) working in the **Shvatka** codebase — an
engine for the night search game *Encounter/Shvatka*, exposing a REST API and a
Telegram bot.

## TL;DR for agents

- **Write new code as `Interactor` classes** (callable, DI-wired), not as free
  service functions. The project is mid-migration — see below.
- **Don't add new to HolderDAO properties** - prefer DI
- **Don't add to middleware data new keys** - prefer DI
- **Don't rewrite existing code** unless the task requires it. Leave working
  service functions alone; only new functionality should adopt the new style.
- **Prefer `IdentityProvider` and `CurrentGameProvider`** for resolving the
  current user/player/team/game everywhere except the DAO layer.
- **New API endpoints → integration tests.** New **domain** classes/methods →
  **unit tests.**
- **Lint and tests run in CI.** You may push to the branch and read the CI
  status instead of running the full (slow, testcontainer-backed) suite
  locally. Running `pytest tests/unit` locally for fast feedback is fine.

## Project layout

```
shvatka/
  core/            # Pure domain + application logic. No framework imports.
    models/        # DTOs (dto.*), enums, action models
    interfaces/    # Protocols: dal/* (DAO contracts), identity, current_game, ...
    services/      # OLD style: free service functions (e.g. game.py, key.py)
    games/         # interactors.py, adapters.py, game_play.py, dto.py
    scenario/      # interactors.py ...
    waiver/        # interactors.py, services.py, adapters.py
    rules/         # pure business rules / checks
  api/             # FastAPI app: routes/, dependencies/, models/ (req + responses)
  tgbot/           # aiogram 3 + aiogram_dialog bot: handlers/, dialogs/, views/
  infrastructure/
    db/            # SQLAlchemy 2 models, dao/ (impls), migrations (alembic)
    di/            # dishka Providers wiring interactors + adapters
    bus/, clients/, scheduler/, picture/, ...
tests/
  unit/            # Fast, no DB. domain/, services/, mapper/, serialization/, ...
  integration/     # Slow, real Postgres via testcontainers. api_full/, bot_full/
  fixtures/, mocks/
```

Dependency direction: `core` knows nothing about `api`, `tgbot`, or
`infrastructure`. The outer layers depend inward. Keep it that way — `core`
talks to the outside world only through Protocols in `core/interfaces/`.

## The Interactor migration (most important convention)

The codebase is evolving **from service functions to `Interactor` classes** with
an async `__call__` and constructor-injected dependencies. New code MUST follow
the Interactor style.

### Old style (don't add more of these; don't gratuitously refactor them)

```python
# shvatka/core/services/game.py
async def upsert_game(
    raw_scn: scn.RawGameScenario,
    author: dto.Player,
    dao: GameUpserter,
    ...
) -> dto.FullGame:
    ...
```

### New style — write this

An Interactor is a class whose dependencies (DAO adapters, providers,
sub-processors) are injected via `__init__`, and whose `__call__` runs the use
case. Two equivalent forms are used; pick whichever fits:

```python
# shvatka/core/games/interactors.py  — plain class
class GameStatReaderInteractor:
    def __init__(self, dao: GameStatReader):
        self.dao = dao

    async def __call__(self, game_id: int, identity: IdentityProvider) -> dto.GameStatWithHints:
        player = await identity.get_required_player()
        game = await self.dao.get_by_id(game_id)
        return await get_game_stat_with_hints(game, player, self.dao)


# or as a frozen dataclass when there are several deps
@dataclass(kw_only=True, slots=True, frozen=True)
class GamePlayReaderInteractor:
    current_game: CurrentGameProvider
    game_play_dao: GamePlayDao

    async def __call__(self, identity: IdentityProvider) -> CurrentHintsAndKeys:
        ...
```

Conventions for Interactors:

- Live next to their domain in `core/<area>/interactors.py`.
- Depend on **Protocols**, not concrete implementations. DAO contracts are
  "adapters" — compose the narrow `core/interfaces/dal/*` protocols into an
  area-specific Protocol in `core/<area>/adapters.py` (see
  `shvatka/core/games/adapters.py`).
- Take `identity: IdentityProvider` (and `current_game` via a constructor dep)
  instead of receiving resolved `player`/`team`/`game` arguments.
- Reuse existing service functions internally where helpful — Interactors often
  wrap them during the migration. That's fine.

### Wiring with dishka

Register interactors and their adapters in `shvatka/infrastructure/di/`
(`interactors.py` and friends). Most interactors register with a bare
`provide(SomeInteractor)`; adapters map a concrete DAO impl onto its Protocol:

```python
class GamePlayProvider(Provider):
    scope = Scope.REQUEST

    @provide
    def get_game_state(self, dao: HolderDao) -> GameStatReader:
        return GameStatReaderImpl(dao)

    get_game_state_interactor = provide(GameStatReaderInteractor)
```

Consume them at the edges via `FromDishka[...]`:

```python
# shvatka/api/routes/game.py
@inject
async def get_game_stat(
    interactor: FromDishka[GameStatReaderInteractor],
    identity: FromDishka[ApiIdentityProvider],
    id_: Annotated[int, Path(alias="id")],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bomzheg/Shvatka](https://github.com/bomzheg/Shvatka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
