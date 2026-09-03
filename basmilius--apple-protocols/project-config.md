---
trigger: always_on
description: TypeScript monorepo voor Apple device protocollen (AirPlay 2, MRP, Companion Link, RAOP). Bun workspace.
---

# Apple Protocols

TypeScript monorepo voor Apple device protocollen (AirPlay 2, MRP, Companion Link, RAOP). Bun workspace.

## Build

```bash
bash build.sh        # Bouwt alle packages in dependency-volgorde
```

Elke package gebruikt `tsgo --noEmit && tsdown` (type-check + bundel). Diagnostics gebruikt `tsgo && bun -b build.ts` (compileert standalone binaries voor 5 platforms).

### Protobuf genereren

```bash
bun --cwd packages/airplay gen:proto   # buf generate → packages/airplay/src/proto/
```

117 `.proto` bestanden in `packages/airplay/proto/`, tooling: `@bufbuild/buf` + `@bufbuild/protoc-gen-es` + `@bufbuild/protobuf`.

## Validatie tegen Homey app

Na elke wijziging moet de Homey app (`~/Development/Projects/homey/com.basmilius.apple`) blijven bouwen:

```bash
# 1. Build apple-protocols
bash build.sh

# 2. Kopieer dist naar Homey node_modules
for pkg in apple-airplay apple-audio-source apple-common apple-companion-link apple-encoding apple-encryption apple-raop apple-rtsp apple-sdk; do
  cp -r "packages/${pkg#apple-}/dist" ~/Development/Projects/homey/com.basmilius.apple/node_modules/@basmilius/${pkg}/dist
done

# 3. Type-check Homey app
cd ~/Development/Projects/homey/com.basmilius.apple && bun run build
```

Als stap 3 faalt, is er een breaking change in de public API.

## Packages (build-volgorde)

| Package                           | Pad                       | Doel                                                                          |
|-----------------------------------|---------------------------|-------------------------------------------------------------------------------|
| `@basmilius/apple-encoding`       | `packages/encoding`       | Plist, OPack, TLV8, DAAP, NTP                                                 |
| `@basmilius/apple-encryption`     | `packages/encryption`     | Ed25519, Curve25519, ChaCha20, HKDF, SRP                                      |
| `@basmilius/apple-common`         | `packages/common`         | Discovery, pairing (HAP M1-M6 + verify), storage, context, mDNS               |
| `@basmilius/apple-audio-source`   | `packages/audio-source`   | Audio decoders: MP3, OGG, WAV, PCM, FFmpeg, URL, SineWave, Live               |
| `@basmilius/apple-rtsp`           | `packages/rtsp`           | RTSP client (request/response, encryption)                                    |
| `@basmilius/apple-airplay`        | `packages/airplay`        | AirPlay 2 protocol: control/data/audio/event streams, 117 protobuf definities |
| `@basmilius/apple-companion-link` | `packages/companion-link` | Companion Link: HID, apps, accounts, power, OPack framing                     |
| `@basmilius/apple-raop`           | `packages/raop`           | RAOP audio streaming via RTSP                                                 |
| `@basmilius/apple-sdk`            | `packages/sdk`            | High-level SDK: AppleTV, HomePod, controllers, discovery, pairing             |
| `@basmilius/apple-diagnostics`    | `packages/diagnostics`    | Interactieve test/debug CLI (standalone binaries)                             |

## Dependency graph

```
encoding          (geen interne deps)
encryption        (geen interne deps)
common            → encoding, encryption
audio-source      → common
rtsp              → common, encoding
airplay           → common, encoding, encryption, rtsp
companion-link    → common, encoding, encryption
raop              → common, encoding, encryption, rtsp
sdk               → airplay, audio-source, common, companion-link, encoding, raop
diagnostics       → sdk + alle protocol packages
```

Alle interne deps gebruiken `workspace:*`. Bij release vervangt CI dit met de release-versie via `sed`.

## Architectuur

```
devices (AppleTV, HomePod)
  ├── airplay/ (AirPlayDevice + Remote, State, Volume, Client, Player)
  │     └── @basmilius/apple-airplay (Protocol, DataStream, ControlStream, AudioStream, EventStream)
  │           └── @basmilius/apple-common (pairing, mDNS, storage)
  │                 ├── @basmilius/apple-encoding
  │                 └── @basmilius/apple-encryption
  ├── companion-link/ (CompanionLinkDevice)
  │     └── @basmilius/apple-companion-link
  └── model/
        ├── AppleTV = AirPlay + CompanionLink (remote control + media + apps + text input)
        ├── HomePod = AirPlay only (media + volume)
        └── HomePodMini = HomePod (zelfde, ander device model)
```

## Key patterns

### Message sending (MRP via AirPlay DataStream)
Berichten worden gebouwd in `packages/airplay/src/dataStreamMessages.ts` en verstuurd via `DataStream.exchange()` (request/response) of `DataStream.send()` (fire-and-forget). Elk bericht is een `ProtocolMessage` wrapper met een protobuf extension.

### State tracking
`packages/devices/src/airplay/state.ts` luistert naar DataStream events en houdt now-playing, volume, keyboard, en output device state bij. `NowPlayingSnapshot` vergelijking voorkomt dubbele events. Consumers luisteren naar State events.

### Now playing hierarchie
`AirPlayState` → `Client` (per bundleIdentifier) → `Player` (per playerPath). Client proxied getters naar de actieve Player. Player extrapoleert `elapsedTime` via Cocoa-timestamp + playbackRate.

### HID events

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basmilius/apple-protocols](https://github.com/basmilius/apple-protocols) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
