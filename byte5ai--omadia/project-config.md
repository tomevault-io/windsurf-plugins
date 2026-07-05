---
trigger: always_on
description: Du bist ein Agent, der an diesem Repo arbeitet. Dieses Dokument ist **nicht optional**. Lies es vollständig, bevor du Code, Config, Secrets, Skills oder Docs änderst.
---

# AGENTS.md — Mandatory Reading for Every LLM Agent Touching This Repo

Du bist ein Agent, der an diesem Repo arbeitet. Dieses Dokument ist **nicht optional**. Lies es vollständig, bevor du Code, Config, Secrets, Skills oder Docs änderst.

Das Projekt wird von **mehreren parallelen Agents** weiterentwickelt (Claude Code in verschiedenen Chats, manchmal mehrere gleichzeitig). Ohne disziplinierte Dokumentation verlieren wir innerhalb von Stunden den Überblick, welcher Zustand produktionswirksam ist. Das ist bereits passiert — siehe `docs/CHANGELOG.md` Eintrag `2026-04-19 — crashloop durch verpasste Schema-Sync`.

## Kernregel: Dokumentieren ist Teil der Aufgabe, nicht ein Nice-to-have

> **Jede Änderung an Code, Schema, Secrets, Skills, Agent-Configs, Dockerfile, fly.toml, Deploy-Pipeline oder Architektur muss im gleichen Arbeitsschritt dokumentiert werden — bevor die Aufgabe als erledigt gilt.**

Ohne Doku-Update ist eine Änderung **nicht fertig**, selbst wenn der Build grün und das Deploy live ist.

## Was wo dokumentiert wird (Entscheidungs-Baum)

| Art der Änderung | Ziel-Dokument | Granularität |
|---|---|---|
| Neue Feature / Architektur-Entscheidung | `docs/middleware-agent-handoff.md` aktualisieren | Abschnitts-Level |
| Bugfix / Ops-Vorfall / Build-Problem | `docs/CHANGELOG.md` Eintrag anhängen | Datum + ein Absatz |
| Security-Entscheidung / Credential-Verschiebung | `docs/security-architecture.md` aktualisieren | Abschnitt |
| Neue ENV-Variable / Secret | `middleware/.env.example` + `docs/middleware-agent-handoff.md` §10 | Zeile + Erklärung |
| Neue Route / Tool / Sub-Agent | `docs/middleware-agent-handoff.md` §3 und §8 | Abschnitt |
| Neue SQL-Migration | Datei in `middleware/src/services/graph/migrations/` — **plus** CHANGELOG-Eintrag mit ID und Zweck | Migration-ID |
| Neue Skill-Version | `skills/<name>/SKILL.md` + CHANGELOG | Skill-Name + Kurzzusammenfassung |
| Offener Punkt / Backlog / TODO | `docs/middleware-agent-handoff.md` §13 Roadmap | Bullet |

Wenn die Zuordnung unklar ist: lieber in CHANGELOG notieren als gar nicht — später konsolidieren.

## Einstiegsreihenfolge für eine neue Session

1. `AGENTS.md` (dieses Dokument)
2. `docs/README.md` — Index aller Docs
3. `docs/middleware-agent-handoff.md` — Architektur + Tech-Stack + Commands
4. `docs/CHANGELOG.md` — zuletzt passierte Änderungen (bremst vor Fehlern, die andere schon hatten)
5. Spezifisches Doc für den aktuellen Task (Security, Graph, Frontend, …)

Ohne mindestens Punkte 1–3 darf kein Code geändert werden.

## Working in a multi-session repo

**Convention (enforced):** the main clone never receives commits. Every change — even a single-line typo fix — lands in a worktree. This is branch-agnostic: an agent whose HEAD got switched to `main` by a parallel session is caught here too, not just one that created a feature branch in the wrong tree. Enforced by the `.hooks/pre-commit` hook shipped via the engineering-standards skill.

~~~bash
git worktree add ../<repo>-<feature> -b <branch> main   # create with new branch
git worktree add ../<repo>-<feature> <existing-branch>  # or attach existing
# work in ../<repo>-<feature>/
git worktree remove ../<repo>-<feature>                 # remove the tree
git branch -D <feature>                                 # remove the branch ref (after merge or discard)
git worktree list                                       # inspect
~~~

Build artefacts (`target/`, `node_modules/`, etc.) live per worktree — first build per tree is full cost, subsequent builds are independent.

**Bypass levels** (in increasing persistence):

| Level | Effect | How |
|---|---|---|
| One-off | this commit only | `ALLOW_MAIN_TREE_BRANCH=1 git commit ...` |
| Per repo | persistent disable, other standards still apply | `git config engineering-standards.main-tree-discipline false` |
| Repo exempt | all engineering-standards disabled for this repo | `status: exempt` in `.github/engineering-standards.yml` |

### Drift signals (for any unusual main-tree work that is allowed)

- Run `git branch --show-current` before each commit and confirm it matches what you intended.
- Treat `git status` anomalies as drift signals: directories you didn't touch showing as `??`, unexpected `M` on files you didn't edit. Don't commit through that — `git reflog | head -20` first to see who moved your HEAD.
- Don't reach for `git reset --hard` reflexively; another session may have uncommitted work in the shared tree. Inspect `git stash list` and `git diff --stat` first.

## Parallele Arbeit — Kollisionen vermeiden

- **Fly-Deploys sind nicht atomar.** Wenn ein anderer Agent gerade deployt, abwarten (30-60s), sonst trittst du ihm auf den Zeh.
- **Secrets-Rotation synchronisieren.** Nicht unangekündigt Secrets überschreiben — ein Agent deployt einen Proxy-Token-Rename, ein anderer Agent hält noch den alten im Skill. Kommuniziere solche Änderungen im CHANGELOG, bevor du die `fly secrets set`-Kommandos tippst.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [byte5ai/omadia](https://github.com/byte5ai/omadia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
