---
trigger: always_on
description: The headless generator (`generator/`) and the web app (`server/`) are both
---

# AGENTS.md

The headless generator (`generator/`) and the web app (`server/`) are both
built and run against the live relay. What has never run is the model call
itself: there is no `ANTHROPIC_API_KEY` in the dev container, so everything from
`Writer.write` onward is untested against the real API. **[`docs/PLAN.md`](docs/PLAN.md) is the design**
— read it before writing code; this file holds only what the plan does not: the
decisions that are settled, the ones that are not, the readings taken off the
live relays, and the conventions to hold to.

## What this is

A service that reads a signed-in Nostr user's web-of-trust view of the last 24
hours and generates a newspaper front page from it, which they then publish to
their own Blossom servers as an nsite.

Sibling project to **[vespa-relay](https://github.com/NosFabrica/vespa-relay)**,
which is the search relay this reads from. That repo's `AGENTS.md` is worth
reading — the commenting conventions, the JitPack pinning trap, and the
"instrument before you theorize" habit all apply here.

## Build

    ./gradlew build                 # compile + test + spotless
    ./gradlew spotlessApply         # run BEFORE committing; formatting alone fails the build
    ./gradlew :generator:installDist
    generator/build/install/generator/bin/generator <npub> --check
    generator/build/install/generator/bin/generator <npub> --dry-run

    ./gradlew :server:installDist
    OBSERVER_INSECURE_COOKIES=true PORT=8099 server/build/install/server/bin/server

`OBSERVER_DB`, `OBSERVER_RELAY`, `OBSERVER_EFFORT`, `PORT`, `HOST` and
`OBSERVER_INSECURE_COOKIES` configure the server. The last one lets the session
cookie travel over plain HTTP and is for local work only — a deployment that
sets it is asserting that TLS terminates somewhere in front of it.

`--check` reports the readiness chain and stops. `--dry-run` does everything
except call the model and writes the digest instead of a page. Neither needs an
API key. A full run reads `ANTHROPIC_API_KEY` from the environment.

## Settled — do not relitigate without a reason

- **There is no fallback lens.** A reader without a resolvable `kind 10040` gets
  the readiness chain and a wait, not a paper. The provisional lens — follows
  plus follows-of-follows, ranked by recency — was built for the 4.5% finding
  and removed on 2026-08-18: it showed a first-time reader the one version of
  the product that cannot demonstrate what the product is for (measured overlap
  with the unranked control: 0 of 400, where a real lens gives 1 of 400), and it
  read up to 120 strangers' follow lists off other people's relays to do it.
  `observer:<pk> sort:rank` with an unresolvable token does not error — it
  silently becomes the anonymous ranking — so the readiness chain is the gate,
  and `Press` refuses with `NO_LENS` rather than building a corpus another way.

- **Nostr goes through quartz. All of it.** `NostrClient` + the `fetchAll` and
  `count` accessories, `Filter`, `Event`, `AdvertisedRelayListEvent`,
  `ServiceProviderTag`, `MetadataEvent`, NIP-19 decoding. The generator once
  carried ~400 lines of hand-rolled bech32, websocket and NIP-01 dispatch; all
  of it already existed in the library the relay itself is built on. What is
  left local is in `nostr/Relays.kt` (timeout and REQ-size policy) and
  `nostr/Tags.kt` (generic tag reads quartz has no named helper for).

- **Window is 24 hours, fixed.** Not "since last login."
- **No prompt caching**, and no shared wire/personal split. Every edition is
  generated standalone from one feed.
- **The model writes the whole document**, markup and optionally CSS. It is not
  filling a schema. Safety is enforced *after* generation by the sanitizer, not
  before it by constraining the writer.
- **No NSFW classification.** The trust provider is the moderator. We honor the
  lens including the parts we would not have chosen.
- **Art is hotlinked**, never fetched, resized, re-hosted or inlined. There is no
  image library in this project.
- **Login required to generate**; no login to read a published edition.
- **The system prompt is fixed, hidden, and never reaches the client.**
- **The paper prints addresses; it does not make them clickable.** No `<a href>`
  to the open web survives — only permalinks back to a source event. An earlier
  version allowlisted any URL that appeared in the corpus; a test caught that
  the corpus is where the attacker writes, so posting a phishing URL was enough
  to allowlist it. Presence in the corpus is evidence of nothing.

## Measured facts about the relays (2026-08-17 — re-measure, do not trust)

- **`search-staging` holds no kind 3 at all**, and `/stats.json` confirms it is
  not a mirrored kind. Follow lists must come from the reader's own write relays,
  discovered from their kind 10002 — which *is* mirrored. The outbox model
  working, not a workaround.
- **NIP-45 COUNT answers** on both `search-staging` and `scores.brainstorm.world`.
  It is still optional, and a null count is a supported answer that must draw
  nothing rather than estimate.
- **`search-staging` sends an AUTH challenge before answering a COUNT**, even
  though `auth_required` is false. Anything resolving on the first non-EVENT
  frame reads the challenge as the answer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NosFabrica/the-nostr-observer](https://github.com/NosFabrica/the-nostr-observer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
