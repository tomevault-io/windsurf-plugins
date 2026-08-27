---
trigger: always_on
description: > This file defines the permanent product and engineering rules for the Liangxiang repository.
---

# AGENTS.md — 梁相 V0.1

> This file defines the permanent product and engineering rules for the Liangxiang repository.
> It applies to Cursor, Codex, and any other coding agent working in this repo.
> If old code, tests, docs, prompts, comments, or mock data conflict with this file, this file wins.
> For the full execution plan, also read `docs/LIANGXIANG_CURSOR_MASTER_R3.md` when present.
> Brand theme and release copy are centralized in `docs/140-liangxiang-brand.md`.

---

## 1. Product Contract

### Product identity

- Product name: **梁相**
- Technical identifiers are exclusively `dsh-liangxiang`, `liangxiang`,
  `/liangxiang/api`, `LIANGXIANG_*`, and `liangxiang-backend`. Do not add
  alternate brand namespaces to code, configuration, storage, routes, logs,
  deployments, tests, or documentation.
- 产品 UI/copy 中，中央角色统一称为 **梁子**；不直接使用现实人物姓名。
- DSH WebUI entry Hover / Focus text: **今日梁相**
- The docked entry's icon IS the current central 梁子 state (one of the five, or
  the 待开梁 placeholder) — never a letter or an abstract logo.
- The entry is freely placeable: it can be dragged anywhere in the frame and the
  position persists per browser (a cosmetic preference, never an authority).
- Panel title: **今日梁案**
- 梁位 = the single public number = global `up_ratio`, printed with decimals.
- There is at most one Active case at a time. Normally one per business date;
  operators may archive today's case and open another on the same date (TEMP).
- Voting is strictly binary:
  - `up` -> UI: **夯**
  - `down` -> UI: **拉**

### Never reintroduce

The following are obsolete and must not return as Liangxiang product concepts:

- `稳`, neutral, steady, abstain, or any third vote option
- candidate / Candidate Ranking
- leaderboard / ranking / winner / Top-N / #1/#2/#3 梁位
- 大夯 / 偏夯 / 胶着 / 偏拉 / 大拉
- global `LiangScore`, 0–100 梁分, Bayesian prior
- `BallotLedger`, `LiangBallot`, 梁签 as the core voting-credit model
- 小难梁 / 老梁
- old personal avatar progression `梁哥 -> 梁总 -> 梁神 -> 梁圣 -> 梁祖`
- personal Token / earned incense / remaining incense driving the central Liangzi state
- “vote must not reduce LiangQi” as an invariant
- cache-read 10% weighting

Do not mechanically delete unrelated generic words from dependencies or third-party code. Remove only obsolete Liangxiang business semantics.

Artwork exception: `牢梁` is intentionally allowed only as the decorative
plaque inside the `WAITING / 待开梁` portrait. It is never a state name, tier,
metric, or UI label. The five active portraits may likewise carry a small
in-art badge/plaque matching their authoritative state name. These embedded
marks are visual jokes, not data or accessible copy.

---

## 2. Frozen UI Structure

The expanded Liangxiang panel has **four visual regions**. Do not add a separate “personal growth tier” section.

### Region 1 — 今日梁案

Show the single current Active case.

Example:

```text
今日梁案
DeepSeek Harness 是夯还是拉
```

### Region 2 — Central core

```text
今日凝香 7 炷     [梁子 + 个人香火环]     下一炷 3,000 当量
                  梁位 83.021952% → 梁神
```

Rules:

- left overlay: personal `earned_incense_today` (`今日凝香` = 今天总共生成的香火)
- center (in-flow, geometrically centered in the panel): concrete Liangzi
  avatar, the 香火环, and the incense-mark overlay. These three MUST share
  the panel's horizontal centerline. Personal flanks are absolutely positioned
  and MUST NOT participate in in-flow width (`space-between` / unequal flex
  columns are forbidden here — 「今日凝香」 being wider than 「下一炷」 must
  never shove 梁子 sideways).
- remaining incense on the ring is pictorial place-value on **separate orbits**
  (炷=stick ones, 月=moon tens, 日=sun hundreds, each 0–9). Glyphs sit on
  almost-full orbits with a bottom gap for the 梁位 pill — not 2px dots on a
  cramped top arc. A moon never occupies a stick slot.
  ≥1000 炷 falls back to a compact numeral on the ring. The ring fill is
  next-incense progress; the 炷/月/日 marks are remaining incense. The two
  flanks are small captions of those same two facts — they must not compete
  with the ring as a second copy of the same information.
- the panel 梁子 and the docked logo bob together. Only the figure layer
  translates; chrome / ring / page stay still. Cadence follows personal
  `liang_qi_fill` (0 = still, approaching 1 = faster), never remaining
  incense or the global 梁位. Respect `prefers-reduced-motion`.
- right overlay: personal `tokens_to_next_incense` in **Pro 当量** (not raw
  Flash Token). Hover/focus on 下一炷 shows the model-weight table
  (Pro ×1 / Flash ×0.5 / 其它 ×0.5).
- visible 下一炷 当量 uses integer compact (`33K`, never `≈ 33.4K`) so the
  flank does not overlap 梁子. Hover title and the screen-reader summary keep
  the exact integer. 今日凝香 / 环上 overflow 仍可用一位小数。
- under the avatar: **exactly one** global number — 梁位 = `up_ratio`, printed with
  6 decimals (`LIANG_POSITION_DECIMALS`) — then a causal arrow to the 称呼
  (`梁位 83.021952% → 梁神`). The authoritative text label remains outside the
  portrait; the artwork may also contain the decorative matching badge allowed
  by the exception above.
- `down_ratio` gets **no** second big number: it is `1 − 梁位` and appears only in
  the tooltip and the screen-reader summary
- the printed 梁位 must be TRUNCATED, never rounded, so it can never read as

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liang-today/dsh-liangxiang](https://github.com/liang-today/dsh-liangxiang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
