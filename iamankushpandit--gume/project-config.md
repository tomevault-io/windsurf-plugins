---
trigger: always_on
description: This project keeps its guidance in **`CLAUDE.md`** files. They apply to everyone who changes this repository — every agent regardless of which tool you are, and every human contributor too. There is one rulebook, not an agent track and a people track: [CONTRIBUTING.md](CONTRIBUTING.md) covers how work reaches the protected `main` and `dev` branches, and these files cover what the code itself has to honour. Read them as your instructions.
---

# Agent instructions

This project keeps its guidance in **`CLAUDE.md`** files. They apply to everyone who changes this repository — every agent regardless of which tool you are, and every human contributor too. There is one rulebook, not an agent track and a people track: [CONTRIBUTING.md](CONTRIBUTING.md) covers how work reaches the protected `main` and `dev` branches, and these files cover what the code itself has to honour. Read them as your instructions.

- [`CLAUDE.md`](CLAUDE.md) — start here: build commands, architecture, invariants, and the protocol for working alongside other agents
- [`src/engine/CLAUDE.md`](src/engine/CLAUDE.md) — screen lifecycle, catalogs, progress tracking
- [`src/games/CLAUDE.md`](src/games/CLAUDE.md) — how to add or change a game
- [`src/hal/CLAUDE.md`](src/hal/CLAUDE.md) — hardware, persistence, profiles, watchdog
- [`src/ui/CLAUDE.md`](src/ui/CLAUDE.md) — theming and drawing helpers

**Modularity rule:** If a file is becoming large (as a rule of thumb, `src/main.cpp` > ~400 lines of active logic in one function, or any `.cpp` > ~600 lines total), **refactor it into a more modular form first** before making the requested change. The refactor must not break existing functionality and must land as its own commit before the feature change that prompted it.

**System/UI app orientation rule:** Any screen that is a system utility (Settings, Wi-Fi, SystemInfo, Profiles, Scores, About, or any future app beyond the playable game catalog) must work in **both landscape and portrait**. Read `tft.width()` / `tft.height()` at render time, not `SCREEN_WIDTH`/`SCREEN_HEIGHT`. Use `Ui::drawTab()` for multi-section layouts; the strip adapts when you divide screen width at render time.

## Two standing rules — you should never have to be told these

**1. No AI attribution in commits, ever -- and no AI in the names either.** Do
not add `Co-Authored-By: Claude`, `Co-Authored-By: <any AI>`, "Generated
with..." footers, or any other trailer or sign-off naming an AI tool or model.
The repository's history records the author, and that is a human. Check your
commit message before you run `git commit` -- this applies to amends, squashes
and PR bodies too.

The same goes for the word itself. No `claude`, and no other model or vendor
name, anywhere your contribution leaves a trace: branch names, commit subjects
and bodies, PR titles and descriptions, file names, identifiers, comments and
TODOs. A branch called `claude/fix-the-thing` says who typed rather than what
changed, and it is permanent in a way the session is not -- it lands in the
merge commit, the pull request and every clone. Name a branch for its work:
`feat/<game-id>`, `fix/<area>`, `docs/<topic>`. Already on one that breaks this?
Rename it before opening the PR (`git branch -m <new-name>`).

The `CLAUDE.md` files are the one exception, because that filename is how you
found these instructions. The rule is about what a *change* carries.

**2. Keep the docs in sync as part of the change, not as a follow-up.** If your
change alters behaviour, structure, dependencies, screens, settings, the game
list or the build, then in the *same* commit you also update whichever of these
it touched:

| You changed | Update |
|---|---|
| Anything user-visible or the feature set | `README.md` — including the games count, the flash/RAM figures from your own `pio run`, and the version |
| Architecture, invariants, layout, build flags, dependencies | `CLAUDE.md` |
| The agent protocol itself | `AGENTS.md` (this file) |
| A subsystem's rules | the `CLAUDE.md` in that directory |
| A new game | `src/games/CLAUDE.md` and the README game table |

Do not wait to be asked, and do not leave it for "a docs pass later". A stale
`README.md` that claims the wrong game count or the wrong flash figure is a
defect, and it is your defect if you shipped the change that made it wrong.

**Run `python tools/check_docs.py` before you commit.** It fails if the version,
the game count, the source-tree listing or the build figures have drifted, and
if the About app has started restating facts instead of deriving them. These
rules existed and the docs went stale anyway -- the README shipped claiming 23
games when there were 26, and the source listing missed three files that had
been added. Asking for vigilance does not work at the end of a long change; the
check does. It is not a substitute for reading the prose, only for the parts a
machine can catch.

If your change alters a screen's layout, or adds or removes one, regenerate the
mock-ups in the same commit: `python tools/gen_screens.py`. `docs/screens/` kept
images of the deleted Countries game for two releases, and the Settings picture
showed a grid that no longer existed. A mock-up of a screen that is not there
any more is a worse lie than a missing one.

## The About app is user-facing documentation — keep it true


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamankushpandit/Gume](https://github.com/iamankushpandit/Gume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
