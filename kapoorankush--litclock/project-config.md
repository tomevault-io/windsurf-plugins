---
trigger: always_on
description: Before every commit, run these in order:
---

# LitClock

## Pre-commit checklist

Before every commit, run these in order:

1. **Lint**: `ruff check src/ image-gen/ tests/ scripts/` (matches CI — full rule set, not just `--select F`)
2. **Tests**: `python3 -m pytest tests/ --ignore=tests/test_eink_display.py -q`
3. **JS tests** (when `node_modules/` is present): `npm run test:js` — covers `src/control_server/static/js/*.js` via vitest + jsdom (#338). Dev/CI only; skip if you haven't run `npm install`. Never required on the Pi.

The eink_display tests require hardware-specific dependencies (Pillow + waveshare display drivers) that aren't available on dev machines. Weather tests now run in CI (astral is lazy-imported inside `is_daytime()`). All non-hardware tests must pass.

Do NOT commit if either step fails.

If the commit touches `image-gen/litclock_annotated.csv`, ship it with `python3 image-gen/corpus_edit.py ship "<message>"` instead of hand-sequencing image regen + `.images-version` bump + release (see CONTRIBUTING.md → "Editing the quote corpus", issue #211).

## Testing

- Framework: pytest (Python) + vitest (JS, control PWA only — #338)
- Python test command: `python3 -m pytest tests/ --ignore=tests/test_eink_display.py -q`
- JS test command: `npm run test:js` (requires `npm install` first; Node 20+)
- Python tests live in `tests/`; JS tests live in `tests/js/`
- Hardware-dependent tests (`test_eink_display.py`) are skipped on dev machines — they need Pillow + waveshare drivers
- JS test infra: vitest + jsdom; loader helpers in `tests/js/helpers/loadScript.js`. See CONTRIBUTING.md → "JavaScript tests" for usage.

## Linting

- Linter: ruff (configured in `pyproject.toml`)
- Check: `ruff check src/ image-gen/ tests/ scripts/` (matches CI)
- Max line length: 120

## First-boot QA checklist

The first-boot flow (`scripts/first-boot.sh`) provisions WiFi via a web UI; everything else (location, timezone, units) auto-populates from IP geolocation after WiFi connects (EPIC #383). Must be tested on real Pi hardware — CI cannot fully simulate hotspot creation, captive portal, IP-geo retry, or e-ink display.

**Critical scenarios to test:**

- **WiFi-only hotspot form**: Verify the setup page shows ONLY the WiFi network picker + password field + Submit button. No Location, Timezone, Temperature, or Mature-content sections — those are PWA-only post-handoff.
- **Hotspot creation**: Power on with no known WiFi networks. Verify the Pi creates a hotspot and displays credentials + QR code on the e-ink screen.
- **Captive portal**: Connect a phone to the hotspot. The setup page should auto-open (or be reachable at the displayed IP).
- **WiFi provisioning**: Select a network from the setup page, enter credentials. Verify the Pi connects and transitions to clock mode.
- **WiFi provisioning failure + retry**: Enter a wrong WiFi password. The page should auto-refresh and show an error banner ("Couldn't join your WiFi…"). Fix the password and resubmit. The Pi should connect on retry. Banner must NOT use the deprecated "home WiFi" phrasing.
- **IP-geo auto-populate (US residential)**: On a US residential WiFi, after submit verify env.sh contains `WEATHER_LATITUDE`, `WEATHER_LONGITUDE`, `WEATHER_LOCATION_NAME` (City, State), `WEATHER_UNITS=imperial`, and `timedatectl` reports the correct timezone — all from one ip-api.com call.
- **IP-geo auto-populate (non-US)**: On a non-US WiFi (or via VPN), verify `WEATHER_UNITS=metric` and tz matches the egress country.
- **IP-geo hard failure**: Block `ip-api.com` (firewall rule or DNS block) during provisioning. Resolver retries 4 times with 1/3/9s backoff. After hard failure: env.sh location keys stay empty, timezone unset. PR2 handoff splash will surface the browser-tz fallback.
- **Connecting splash (PR2)**: After submitting WiFi creds, the e-ink should swap the hotspot QR for a "Connecting to {SSID}…" splash while WiFi joins + IP-geo runs (~30s), before the handoff splash.
- **Handoff splash + quote gate (PR2)**: After IP-geo succeeds, the e-ink shows the "Ready to read." handoff splash (settings summary block: Location/Timezone/Units/Mature + PWA QR top-right encoding `http://<IP>` — port 80, no port shown, #343). Quotes must NOT start yet — `litclock.service` is gated on `/etc/litclock/.handoff-complete`. Verify the QR scans to the PWA and the Status tab shows the "Setup complete" top-sheet banner with a "Done — Start the Clock" button.
- **Handoff completion paths (PR2)**: Verify ALL of: (a) tapping "Done" in the PWA, (b) saving any setting in PWA Settings, (c) waiting 120s — each writes `.handoff-complete` and the e-ink paints the first quote within ~1 min. The AtHS hint must stay suppressed while the banner is up, then appear after Done.
- **Handoff failure (IP-geo blocked, PR2)**: With `ip-api.com` blocked, the e-ink shows "Almost ready." + "Not detected" rows; the PWA banner shows "Almost there." with a "Use {browser-tz}" button. Tapping it sets the system tz and starts quotes. Quotes must NOT start on the 120s timer when tz is unknown (a wrong-time clock is worse than no clock).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kapoorankush/litclock](https://github.com/kapoorankush/litclock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
