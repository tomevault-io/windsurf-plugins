---
trigger: always_on
description: Go backend (SQLite, `go:embed`s the built SvelteKit SPA) + SvelteKit 5 admin UI
---

# Calnode — agent notes

Go backend (SQLite, `go:embed`s the built SvelteKit SPA) + SvelteKit 5 admin UI
served under `/admin/`. Public booking pages are server-rendered Go templates
(`internal/handler/templates/*.html`), distinct from the Svelte admin app.

## Frontend toolchain

- Vite 8 (Rolldown) · SvelteKit 2 · Svelte 5 · Tailwind v4 (`@tailwindcss/vite`)
  · bits-ui / shadcn-svelte · Vitest 4 browser mode.
- The frontend is embedded at Go compile time (`frontend/embed.go` →
  `//go:embed all:build`). To see frontend changes in the running app: rebuild
  the frontend (`pnpm build` in `frontend/`) **and** rebuild/restart the Go
  binary — restarting Go alone won't pick up new assets.

## UI styling — required check

**Default to shadcn-svelte in the admin UI.** In the SvelteKit admin app (`frontend/`),
build from the existing shadcn-svelte components — `Button`/`buttonVariants`,
`ConfirmDialog` (**never** `window.confirm`/`alert`), `Dialog`, `Input`, `Switch`,
`Tooltip`, etc. Don't hand-roll buttons, modals, or browser-native dialogs. Destructive
actions use `ConfirmDialog` with `destructive`; row actions use a ghost icon button +
`Tooltip` (see `event-types`, `members`, `recordings`). **If shadcn genuinely doesn't fit,
flag it (and the reason) before deviating — don't silently hand-roll.** This does NOT apply
to the public booking templates (`internal/handler/templates/*.html`), `embed.js`, or the
LiveKit room — those are intentionally framework-free (Go templates / vanilla JS, own CSS).

shadcn-svelte components style state via Tailwind `data-*` variants. Bits-ui
states exposed as `data-state="…"` (checked/unchecked/open/closed) need an
`@custom-variant` remap in `frontend/src/app.css` or they render **silently
unstyled** (logic works, visuals don't). See `frontend/TESTING.md`.

**After changing `frontend/src/lib/components/ui/**`, `frontend/src/app.css`, or
the theme — run `pnpm test:visual`** (Vitest browser smoke). Unit tests do NOT
catch this class of bug; only the real-browser computed-style assertions do.

## Booking calendar — THREE surfaces, keep them aligned

The date/time-slot booking calendar exists in **three** places. A change to its
behaviour or markup must usually be made in all three, or they drift:

1. **Booking page** — `internal/handler/templates/book.html` (server-rendered Go template + vanilla JS)
2. **Manage page** — `internal/handler/templates/manage.html` (reschedule flow; same calendar/slots)
3. **Embed widget** — `internal/handler/embed.js` (Shadow-DOM Web Component on customer sites)

- **Styling is shared:** all three load `internal/handler/templates/booking.css`
  (served at `GET /booking.css`; the widget injects it into its shadow root). Change
  visuals **there**, once — don't re-style per surface.
- **Markup + JS are NOT shared** (Go template vs web component): the calendar render,
  slot picking, and the **mobile step-flow** (calendar → slots → form, with Back) are
  implemented separately in each. If you change calendar *behaviour*, update all three.
- Verify on **desktop and mobile** for each surface after touching the calendar.

## Conversational booking assistant (optional LLM layer)

The "Book by chat" assistant lives on **two** of those surfaces — `book.html` (floating
drawer + inline link) and `embed.js` (inline link only; no floating button, to avoid
colliding with host-site widgets). **Not** on the manage page (reschedule context — a
reschedule chat is deliberately deferred). Server side is one endpoint,
`POST /v1/event-types/{slug}/assistant` (`booking_assistant.go`): an LLM tool-loop that
drives `find_available_slots`/`book` over the **shared deterministic cores** (`computeSlots`,
`createBookingForSlug`) — never re-implement booking logic in the assistant. Invariants:
the LLM does NL→constraints only (never time math), sees only computed availability (never
raw calendar data), and `<think>` reasoning is stripped. Shared `.asst-*` styles are in
`booking.css`; the base prompt (`assistantBaseRules`) is code-owned, admins only append
"Additional instructions". Off by default — `getLLM()` nil → the picker is the fallback.

## Built-in video meetings (LiveKit)

Self-hostable video as a booking location type (`location_type = "livekit"`). **No LiveKit SDK
server-side** — all tokens are hand-signed. The browser room app is **vanilla JS + a vendored
client SDK**, not Svelte.

- **Where it lives:** room UI = `internal/handler/templates/livekit-room.html` +
  `assets/livekit-room.js` (+ vendored `assets/livekit-client.umd.min.js`). Server =
  `internal/livekit/` (`livekit.go` token signing, `admin.go` Twirp/egress) and
  `internal/handler/livekit_room.go` + `livekit_recording.go`. Settings UI =
  `frontend/src/routes/settings/video/`.
- **Three token kinds (don't conflate):** (1) **room token** — opaque HMAC blob in the join URL
  (`{r,e,role}`), carries no LiveKit grant; (2) **access token** — the real LiveKit HS256 JWT the
  SDK joins with (`AccessToken`/`VerifyAccessToken`); (3) **admin token** — short-lived JWT for
  Twirp server APIs.
- **Host authority — `authorizeHost`, NOT just the room token.** A host action is allowed if the
  caller is the **durable host** (`hostRoomOrOwner`: holds a host room token OR is the signed-in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Calnode/calnode](https://github.com/Calnode/calnode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
