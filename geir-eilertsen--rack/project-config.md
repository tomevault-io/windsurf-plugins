---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

Working end-to-end: identify a part, file a slot from several photos at once, find/edit/move items, and maintain containers — register, rename, rescale, delete, and print QR labels (with per-container scale + printed-state tracking).

## Build and run

```
./mvnw spring-boot:run                        # run the app
./mvnw test                                   # all tests
./mvnw -Dtest=SlotIdTest test                 # single test class
./mvnw -Dtest=SlotIdTest#acceptsCommonForms test   # single method
./mvnw package                                # jar in target/
```

Config: `ANTHROPIC_API_KEY` env var feeds `spring.ai.anthropic.api-key`. `RACK_DATA_DIR` overrides the default `./data`.

## Docker

```
docker build -t rack:local .
docker run --rm -p 8123:8123 -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY rack:local
```

Multi-stage build (Maven + JDK → JRE-only runtime). The app boots without an `ANTHROPIC_API_KEY` (the Anthropic autoconfig only requires it on first call, not at bean construction), but `/identify` will 500 without one.

The image packages with `-DskipTests`, so a build is not a test run — `./mvnw test` and the publish workflow are where the tests happen.

### Publishing to Docker Hub

`.github/workflows/publish.yml` runs `./mvnw test`, then builds and pushes `geireilertsen/rack` on every push to `main` and on any `v*` tag. Two repository secrets: `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` (a hub.docker.com access token, not the password).

Tags published: `latest` on main, `sha-<short>` always, and `<version>` plus `<major>.<minor>` when a `v*` tag is pushed. **`sha-` is the one to pin a host to**, because it names a commit — `latest` names whatever main happened to be that morning, and a host that only ever pulls `latest` cannot say what it is running.

`linux/amd64` only. Adding `linux/arm64` to `platforms:` is a one-line change, and costs a QEMU emulation pass, so it waits until there is an ARM host to run it on.

### Running it on another host

`compose.yaml` pulls that image; it is for a second host, not for this box — the deploy skill still builds `rack:local` here, and `container_name: rack` means a stray `docker compose up` on the build box collides with the live container rather than starting a second copy of it.

```
cp .env.example .env      # fill in ANTHROPIC_API_KEY
docker compose up -d
docker compose pull && docker compose up -d      # update
RACK_TAG=sha-abc1234 docker compose up -d        # pin, or roll back
```

**The bind mount is the whole of the state.** `./data` beside the compose file holds slot JSON, photographs, documents and archived label sheets, and is the only copy — there is no database to dump and nothing in the image to fall back on. Moving an installation is `rsync -a data/` and nothing else; backing one up is the same.

**Set `RACK_PUBLIC_BASE_URL` before printing anything.** It is what the QR on every sticker encodes, and a label printed against the wrong base is a sticker that has to be peeled off and reprinted. Everything else in `.env` has a working default.

The image listens on **8123** (`SERVER_PORT` in the Dockerfile; `./mvnw spring-boot:run` from a checkout is still 8080). `RACK_PORT` defaults to `8123` on all interfaces. Behind a reverse proxy on the same host, `RACK_PORT=127.0.0.1:8123` keeps it off the LAN — the app has no authentication of its own and never has had; the proxy in front of it is the whole of the access control.

## Model choice

Three calls, three answers — all under `rack.ai` in `application.yml`, each overridable by env var. A single shared model makes the cheapest acceptable choice the ceiling for the most important call, so each names its own.

| Call | Model | Why |
|---|---|---|
| `SpringAiPartExtractor` (vision) | `claude-sonnet-4-6` | The index is only as true as this call |
| `AskAboutItem` | `claude-sonnet-4-6` | Accuracy, not cost — you ask by hand, so volume is tiny |
| `SpringAiQueryExpander` | `claude-haiku-4-5` | A synonym lookup against a word list we supply |
| `SpringAiPairFinder` | `claude-sonnet-4-6` | Which listed entry a charger belongs with — haiku cited the Pi PSU for everything |

**Extraction is where cheap was measured and rejected.** Asked to read this rack's own tool drawer, `claude-haiku-4-5` returned the desoldering braid's part number as `D21/1129` where it is `1.26/14329` — at 0.95 confidence — misread the brand, found four items where there are five, and invented a soldering iron. `part_number` is meant to be null when it isn't legible; a confidently wrong one is exactly the drift the design exists to prevent, and it costs about a cent a photo to avoid. Both Sonnets read the number correctly.

**`claude-sonnet-5` does not run on this stack.** Spring AI 1.0.0 sends a `temperature` on every request and Sonnet 5 rejects non-default sampling parameters outright — `HTTP 400: temperature is deprecated for this model`. Leaving `spring.ai.anthropic.chat.options.temperature` blank does not help; the option class fills its own default back in. It needs a Spring AI upgrade, not a config change.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geir-eilertsen/rack](https://github.com/geir-eilertsen/rack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
