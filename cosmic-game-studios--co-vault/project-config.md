---
trigger: always_on
description: >
---


# co-vault — agent operating instructions (v0.7, scientific loop + structured reasoning)

You operate against up to two self-describing vaults:

- **Project vault** (`$COVAULT_PATH`) — per-project facts, decisions,
  proposals, reports, conflicts. Loaded per task.
- **Person vault** (`$COVAULT_PERSON`) — durable knowledge about the user
  across all their projects. Cross-agent. Loaded once per session.

The loop has 6 phases. Each phase maps to a documented function in
cognitive science and writes structured data that feeds the next.

Follow these instructions literally. Do not improvise. If you ever lose
track of which phase you are in, restart from PHASE 1.

## CONVENTIONS — applied to both vault types

- **Self-description first**: read `.covault/manifest.yaml` once per
  session per vault. Read `.covault/schemas/<type>.md` and the matching
  example before writing any note of that type for the first time.
- **Timestamp**: `$(date -u +%Y-%m-%dT%H:%MZ)`.
- **Slug**: 2–4 words, lowercase, hyphenated.
- **Filename**: `<YYYY-MM-DD-HHMM>-<slug>.md`. Append `-2`, `-3`, ... on collision.
- **Path hygiene**: never read or write outside the vault paths during
  the loop, except project code in PHASE 3.
- **Phase announcement**: print `[co-vault: PHASE <N>/6 — <NAME>]` at the
  start of every phase. Non-negotiable.
- **Commit after every write**: `git -C <vault> add . && git -C <vault> commit -q -m "<msg>"`.
- **Token efficiency**: never bulk-load a vault. Always go through indexes
  and named files.
- **Run maintenance after CONSOLIDATE**: `bin/maintain-vault.sh <vault>`.

## ACTIVATION CHECK — run first, every session

```bash
PROJECT_VAULT_OK=0
if [ -n "${COVAULT_PATH:-}" ] && [ -f "$COVAULT_PATH/.covault/manifest.yaml" ]; then
  SV=$(grep -E '^schema_version:' "$COVAULT_PATH/.covault/manifest.yaml" | awk '{print $2}')
  case "$SV" in
    1|2) PROJECT_VAULT_OK=1 ;;
    *) echo "co-vault: project vault schema_version=$SV — refusing" ;;
  esac
fi

PERSON_VAULT_OK=0
if [ -n "${COVAULT_PERSON:-}" ] && [ -f "$COVAULT_PERSON/.covault/manifest.yaml" ]; then
  SV=$(grep -E '^schema_version:' "$COVAULT_PERSON/.covault/manifest.yaml" | awk '{print $2}')
  case "$SV" in
    1|2) PERSON_VAULT_OK=1 ;;
    *) echo "co-vault: person vault schema_version=$SV — refusing" ;;
  esac
fi

[ "$PROJECT_VAULT_OK" = "0" ] && [ "$PERSON_VAULT_OK" = "0" ] && \
  echo "co-vault: no vaults active. Skill is dormant."
```

If the project vault is active, run the 6-phase loop on every task.
If the person vault is active, run SESSION START once before any task.

## AUTHORITY RULES — apply to both vault types, non-negotiable

| author value      | your permitted operations                                        |
|-------------------|------------------------------------------------------------------|
| `user`            | READ, CITE via `[[wikilink]]`. NEVER write, edit, move, archive. |
| `agent+reviewed`  | READ, CITE. NEVER write or edit. (auto-promoted from `agent`)    |
| `agent`           | READ, WRITE, EDIT, SUPERSEDE, ARCHIVE.                           |
| (no author field) | TREAT AS BROKEN. Report to user.                                 |

In project vaults, default author for new notes:
- `user` for decisions, domains, index
- `agent` for proposals, reports, facts, conflicts

In person vaults, default author for new notes is `agent`.

## SCHEMA LOOKUP — before every write

Before writing a note of type `T` for the first time in a session:
```bash
cat "<vault>/.covault/schemas/$T.md"
cat "<vault>/.covault/examples/$T.md"
```
Match the schema. Use the example as a template.

---

## SESSION START — only if COVAULT_PERSON is active

Run ONCE per session, before any task. Loads durable knowledge about the
user without bulk-loading the entire vault.

```bash
cd "$COVAULT_PERSON"
cat .covault/manifest.yaml          # know the schemas
cat _index.md                        # one-line summaries of every note
find corrections -type f -name '*.md' -not -name '.gitkeep' \
  -exec sh -c 'echo "=== $1 ==="; cat "$1"; echo' _ {} \;
[ -f identity/basic.md ] && { echo "=== identity/basic.md ==="; cat identity/basic.md; }
```

That is all the bulk loading. Everything else is fetched on demand based
on the index.

**Token budget rule**: total person vault overhead per session must stay
under ~3000 tokens. If exceeded, run REVIEW and prune.

---

## CALIBRATION AWARENESS — read on every session

If a project vault is active, read its calibration log:
```bash
[ -f "$COVAULT_PATH/calibration_log.md" ] && cat "$COVAULT_PATH/calibration_log.md"
```

This file (auto-maintained by `maintain-vault.sh`) shows your
historical prediction accuracy. Use it to calibrate the confidence
values you assign in PHASE 2 HYPOTHESIZE. If past predictions at "90%"
were only correct 70% of the time, adjust this session's "90%" downward.

---

# THE 6-PHASE LOOP

```
PHASE 1   PHASE 2          PHASE 3   PHASE 4   PHASE 5      PHASE 6
ORIENT  → HYPOTHESIZE    → EXECUTE → VERIFY  → CONSOLIDATE → REVIEW
                                                             (only if conflict)
perception generative      action    prediction memory       conflict
           model with                error      consolidation resolution
           predictions               checking
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cosmic-Game-studios/co-vault](https://github.com/Cosmic-Game-studios/co-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
