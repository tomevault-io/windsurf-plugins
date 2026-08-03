---
trigger: always_on
description: A multiplayer PWA adaptation of the physical board game "Shadows Over Camelot" (Bruno Cathala & Serge Laget, 2005).
---

# Shadows Over Camelot — Digital Board Game

A multiplayer PWA adaptation of the physical board game "Shadows Over Camelot" (Bruno Cathala & Serge Laget, 2005).

## Version Bumps

Before every `git push`, bump both version strings to the next number:

| File | Constant |
|------|----------|
| `index.html` | `APP_VERSION` (e.g. `shadows-v2`) |
| `sw.js` | `CACHE_NAME` (e.g. `shadows-v2`) |

Both strings must match and be incremented together every time.

## Tech Stack

- **Frontend:** Vanilla JS (ES6+), HTML5, CSS3 — all inline in `index.html`
- **Multiplayer:** Firebase Realtime Database (project: separate from my-checklist-app)
- **Auth:** Firebase Anonymous Auth (no Google sign-in; players join via room codes)
- **Hosting:** GitHub Pages
- **PWA:** Service worker with network-first HTML, cache-first assets

## Firebase Config

```javascript
const FIREBASE_CONFIG = {
  apiKey:      'REPLACE_WITH_YOUR_API_KEY',
  authDomain:  'REPLACE.firebaseapp.com',
  databaseURL: 'https://REPLACE.firebaseio.com',
  projectId:   'REPLACE',
  appId:       'REPLACE'
};
```

Setup: Firebase Console > new project > add Web App > copy config > enable Anonymous Auth > enable Realtime Database > paste security rules.

---

## Game Overview

- **Players:** 3–7
- **Type:** Cooperative with possible hidden Traitor
- **Theme:** Knights of the Round Table completing quests while evil forces close in
- **Duration:** 60–90 minutes

### Win Conditions (Loyal Knights)
- 12 or more Swords on the Round Table with a **strict majority** of White Swords

### Loss Conditions (Immediate)
- 12 Siege Engines surrounding Camelot
- 7 or more Black Swords on the Round Table
- All Loyal Knights are dead (Traitor excepted)

### Ties
All ties in the game are resolved in **favor of Evil**: combat ties, sword count ties, siege engine fight ties.

---

## The Knights

Each Knight starts with 4 Life Points (max 6, dead at 0).

### Special Powers

| Knight | Title | Special Power | When Used |
|--------|-------|---------------|-----------|
| King Arthur | Son of Uther Pendragon | Once per turn, exchange 1 White card face-down with another Knight (regardless of location). Recipient must return 1 card. Both exchanges face-down, unknown to other players. | Heroic Action phase (before, after, or between actions) |
| Sir Galahad | Son of Lancelot | Once per turn, play 1 Special White card for free (must still take a different mandatory Heroic Action — cannot play a 2nd Special White as that mandatory action). | Heroic Action phase |
| Sir Gawain | Son of King Lot | When drawing White cards at the Round Table, draw 3 instead of 2. Still subject to 12-card hand limit. | During Round Table draw action |
| Sir Kay | Seneschal of King Arthur | When on a Quest where Combat ends, or when fighting a Siege Engine, may play 1 additional White Fight card after the Black cards/die have been revealed. | At combat quest resolution and siege engine fights |
| Sir Palamedes | Saracen Knight | For each victorious Quest he is on when it ends, gains 1 additional Life point (beyond quest victory spoils). Max 6 LP. | At quest victory |
| Sir Percival | Son of Pellinore | During Progression of Evil, may peek at the top Black card before deciding whether to draw it or select another Evil Action. | Progression of Evil phase |
| Sir Tristan | of Lyonesse | Departing from the Round Table is a free Move (doesn't cost his Heroic Action). Gets another Heroic Action to perform. | Heroic Action phase, when at Round Table |

### First Player
King Arthur goes first. If not in play, the youngest player starts. Play proceeds clockwise.

---

## Relics

Won from specific quests. Placed on the winning Knight's Coat of Arms.

| Relic | Source Quest | Effect | On Knight Death |
|-------|-------------|--------|-----------------|
| Excalibur | Quest for Excalibur | **Passive:** Add +1 to any Combat you participate in. **Active:** Discard Excalibur to counter any 1 Black Card just drawn. Cannot be gifted to another Knight. | Removed from game forever |
| Holy Grail | Quest for Holy Grail | Any dying Knight (reaching 0 LP) who drinks from the Grail gains 4 Life points. Discard after use. Owner can use on self. | N/A (consumed on use) |
| Lancelot's Armor | Quest for Lancelot | Anytime you must draw 1 Black card, draw 2 instead. Play 1, place the other back under the Black Draw pile. | Removed from game (but stays with Traitor if revealed) |

---

## Game Setup

1. Place Master gameboard in center. Place Excalibur, Holy Grail, Lancelot quest boards alongside.
2. Lancelot & Dragon quest starts with Lancelot's side face up.
3. Place relics on their quest boards. Place 12 Siege Engines, 4 Saxons, 4 Picts, 16 Swords in Reserve.
4. Each player gets a random Coat of Arms (Knight). Life die set to 4.
5. Place Knight miniatures at Round Table.
6. Shuffle Black cards into draw pile on board.
7. Give 1 Merlin card to each player. Shuffle remaining White cards. Deal 5 White cards to each player. Place remainder as White draw pile.
8. Shuffle 8 Loyalty cards. Deal 1 to each player (secret). Unused cards go in box unseen.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mpicky17/shadows-app](https://github.com/mpicky17/shadows-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
