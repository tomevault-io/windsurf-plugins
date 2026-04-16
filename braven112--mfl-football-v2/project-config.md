---
trigger: always_on
description: ALL features, utilities, and data structures should be designed with the **Auction Price Predictor** in mind. Every function must be **reusable** and **composable**.
---

# MFL Football v2

## Development Principle

ALL features, utilities, and data structures should be designed with the **Auction Price Predictor** in mind. Every function must be **reusable** and **composable**.

---

## Year Rollover System

Two critical dates drive year transitions:

| Date | Event | What Changes |
|------|-------|--------------|
| **Feb 14th @ 8:45 PT** | New MFL league created | `getCurrentLeagueYear()` updates |
| **Labor Day** | NFL season starts | `getCurrentSeasonYear()` updates |

### Decision Framework

**Use `getCurrentLeagueYear()`** for:
- Rosters, contracts, salary cap, auctions, trade analysis
- Key question: *"Does this page help manage my roster?"*

**Use `getCurrentSeasonYear()`** for:
- Standings, playoffs, MVP tracking, draft order
- Key question: *"Does this page show results from games played?"*

```typescript
import { getCurrentLeagueYear, getCurrentSeasonYear, getNextDraftYear, getNextAuctionYear } from '../utils/league-year';
```

Test date-dependent features with `?testDate=YYYY-MM-DD` URL parameter.

---

## Team Name Display

**CRITICAL:** Always use `chooseTeamName()` to prevent UI overflow:

```typescript
import { chooseTeamName } from '../utils/team-names';

const displayName = chooseTeamName({
  fullName: team.name,
  nameMedium: assets?.nameMedium,  // ≤15 chars (default)
  nameShort: assets?.nameShort,    // ≤10 chars
  abbrev: assets?.abbrev,          // 2-6 chars
}, 'default'); // Context: 'default' | 'short' | 'abbrev'
```

Config locations:
- TheLeague: `src/data/theleague.config.json`
- AFL Fantasy: `data/afl-fantasy/afl.config.json`

---

## Editorial Design Standard

**CRITICAL:** All new pages and components must follow the **editorial design language** derived from the PlayerDetailsModal system. This is the visual standard for the entire site.

**Canonical reference:** `src/components/theleague/PlayerDetailsModal.astro`
**Full pattern catalog:** [design-system.md](docs/claude/insights/domains/design-system.md) (search "Editorial Design Standard")
**Component guide:** [components.md](docs/claude/components.md) (Editorial Design Standard section)

### Signature Patterns

**Section titles** — Uppercase, left-border accent:
```css
font-size: 0.75rem;
font-weight: 700;
text-transform: uppercase;
letter-spacing: 0.06em;
padding-left: 0.625rem;
border-left: 2px solid var(--color-primary, #1c497c);
```

**Section titles with subtitles** — When a section title needs a description, wrap both in `.section-header` so the left-border spans both lines:
```html
<div class="section-header">
  <h3 class="section-header__title">NFL Analysis</h3>
  <p class="section-header__sub">Players on the same NFL team</p>
</div>
```
The subtitle is `0.8125rem`, `gray-400`, with `0.25rem` gap. Use standalone section title (above) when there's no subtitle.

**Detail rows** — Flex rows with fixed-width uppercase labels (gray-400) + flexible values, separated by gray-50 borders

**Key metrics** — 3-column grid, gray-50 bg cards, large tabular-nums values + micro uppercase labels

**Tables** — Sticky uppercase headers (0.625rem, gray-400), hover rows, tabular-nums for numbers

**Numbers** — Always `font-variant-numeric: tabular-nums`

**Defensive CSS** — Always include token fallbacks: `var(--color-gray-700, #374151)`

**Modal/overlay backdrop** — **CRITICAL:** Every modal and overlay MUST use the frosted-glass backdrop blur. Never use a plain dark overlay without blur:
```css
background: rgba(15, 23, 42, 0.45);
backdrop-filter: blur(2px);
```

### Typography Scale

| Role | Size | Weight | Notes |
|------|------|--------|-------|
| Hero/page title | 1.35rem | 700 | line-height: 1.2 |
| Section title | 0.75rem | 700 | UPPERCASE + left border |
| Body/values | 0.875rem | 400–500 | |
| Detail label | 0.75rem | 600 | UPPERCASE, gray-400 |
| Table header | 0.625rem | 600 | UPPERCASE, gray-400 |

---

## Player Display (Player Lockup)

**CRITICAL:** Whenever displaying players in a list, card, or table, use the standard **Player Lockup** pattern for consistency:

### Layout
| Left | Right (Row 1) | Right (Row 2) |
|------|---------------|---------------|
| Circular headshot (spans 2 rows) | **Player name** (bold) | NFL team logo (16px) + Position |

### Astro Component
Use `PlayerCell.astro` — it handles DEF position, team code normalization, and optional modal support automatically:

```astro
import PlayerCell from '../components/theleague/PlayerCell.astro';

<!-- Basic usage (DEF handling is automatic) -->
<PlayerCell name={player.name} headshot={player.headshot} position={player.position} nflTeam={player.nflTeam} />

<!-- Compact size -->
<PlayerCell name={player.name} headshot={player.headshot} position={player.position} nflTeam={player.nflTeam} size="compact" />

<!-- With modal support + badges -->
<PlayerCell name={player.name} headshot={player.headshot} position={player.position} nflTeam={player.nflTeam} playerData={playerObj}>
  <span slot="after-name" class="injury-badge">Q</span>
</PlayerCell>
```

### JS Utility (for client-side rendering)
Use `buildPlayerCellHTML()` when building HTML strings in `<script>` tags:

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/braven112) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
