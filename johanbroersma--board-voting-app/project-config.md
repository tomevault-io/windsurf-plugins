---
trigger: always_on
description: A browser-based voting application for generic board elections (e.g. NRCEA school board). Runs on a local Python server; no external services or npm/frameworks. Forked from Church Voting App — all church-specific fields (elder/deacon, congregation name) have been removed.
---

# Board Voting App — Project Guide

## Project Summary
A browser-based voting application for generic board elections (e.g. NRCEA school board). Runs on a local Python server; no external services or npm/frameworks. Forked from Church Voting App — all church-specific fields (elder/deacon, congregation name) have been removed.

## Files
| File | Purpose |
|---|---|
| `server.py` | Python HTTP server — serves static files + `/api/state` + `/api/ballot` |
| `index.html` | Admin hub — Setup, Round Control, Paper Ballot Entry, Election Dashboard |
| `vote.html` | Voter page — token entry → ballot → done/waiting states |
| `election_state.json` | Shared election state, created automatically on first save |
| `build_manual.py` | Generates `manual.docx` using python-docx (no pip deps beyond python-docx) |
| `manual.docx` | Word document user manual |
| `manual.html` | HTML version of the manual |
| `render.yaml` | Render.com deployment config |
| `requirements.txt` | No external deps (stdlib only for server.py) |

## Running Locally
```bash
python3 server.py          # default port 8080
python3 server.py 9000     # custom port
```
- Admin: `http://localhost:8080/`
- Voters: `http://<laptop-ip>:8080/vote.html`

## Architecture

### State Management
- All state lives in `election_state.json` on the server
- `localStorage` key `bva_election_v1` is written as a fallback but is not the source of truth
- `load()` in both HTML files: tries `GET /api/state` first, falls back to localStorage
- `save()` in `index.html`: POSTs full state to `POST /api/state` (admin-authenticated)
- `save()` in `vote.html`: **no-op** — voters never write full state
- Votes are submitted via `POST /api/ballot` (atomic, server-side, thread-locked)

### API Endpoints
| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/state` | GET | None | Read full election state |
| `/api/state` | POST | `adminPasswordHash` in payload must match stored hash | Full state overwrite (admin) |
| `/api/ballot` | POST | Valid token code + round | Atomic ballot submission (voters) |

### Security on Public Hosting
`POST /api/state` validates that the `adminPasswordHash` in the incoming payload matches the stored hash. The first write (empty state) is accepted unconditionally.

### Passwords
- Six passwords: **Landing**, **Admin**, **Election**, **Results**, **Tokens**, **Paper Ballot**
- Stored as SHA-256 hashes in state
- Admin default: `boardvoting`; Results default: `results2024`
- `hashPw()` uses `crypto.subtle.digest` with a pure-JS `sha256Fallback()` for plain HTTP contexts

### Elections
- Single `election` object — no elder/deacon distinction
- Key fields: `nominees`, `openPositions`, `votesPerVoter`, `currentRound`, `votingOpen`, `candidates`, `ballots`, `paperBallots`, `rounds`, `electedAll`, `complete`
- Token structure: `{ code, usedRounds: [] }` — flat list of round numbers already voted
- `org` (Organization Name) and `electionName` replace old church-specific fields

### Auto-refresh Intervals (index.html)
| Screen | Interval | What updates |
|---|---|---|
| Round Control | 3s | Vote counts + participation stats |
| Paper Ballot Entry | 4s | Ballot log only (form selections preserved) |
| Election Dashboard | 3s | Full re-render |

### Voter Page States (vote.html)
`'token' | 'election-ballot' | 'election-done' | 'election-complete' | 'waiting' | 'idle'`
- `waitingPoller`: 3s interval in `waiting / idle / token / election-complete` states
- `donePoller`: 5s interval after vote submitted — notifies voter when next round opens
- Date display fix: `state.meetingDate` is parsed as local date parts (not `new Date(string)`) to avoid UTC timezone offset shifting the day

## Key Functions

### index.html
| Function | Purpose |
|---|---|
| `defaultState()` | Returns clean state object with all defaults |
| `deepMerge(target, source)` | Recursively merges saved state onto defaults |
| `migrateState(state)` | Upgrades old church-app state (elder/deacon → election, nested usedRounds → flat) |
| `load()` | GET `/api/state`, fallback to localStorage |
| `save(state)` | POST `/api/state`, mirror to localStorage |
| `hashPw(pw)` | Async SHA-256 (WebCrypto or fallback) |
| `electionColor()` | Returns color palette object `{dark, mid, tint, border, bar}` |
| `electionLabel(state)` | Returns `state.electionName` or `'Board Election'` |
| `electionBadge(state)` | Returns badge HTML span |
| `countBallotVotes(state)` | Returns `{total, digital, paper, absentee}` for current round |
| `buildResults(state)` | Returns sorted candidate array for current round |
| `renderRoundControl()` | Renders active round screen + starts 3s poll |
| `renderRCResults(state)` | Updates candidate vote bars |
| `renderRoundTransition(state)` | Post-round transition UI |
| `rtCompleteElection()` | Marks election complete, navigates to election-complete screen |
| `printTokenCards()` | Generates fresh QR via `getQrDataUrl(url)`, renders print grid |
| `getQrDataUrl(url)` | Generates QR into hidden off-screen element (not from DOM) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johanbroersma/Board-Voting-App](https://github.com/johanbroersma/Board-Voting-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
