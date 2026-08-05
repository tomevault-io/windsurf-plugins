---
trigger: always_on
description: Sandpolis is a Rust-based virtual estate manager that covers:
---

Sandpolis is a Rust-based virtual estate manager that covers:

- Physical devices like servers, desktops, phones, etc
- Purely virtual entities like accounts, EC2 instances, etc

It's comprised of multiple applications:

- Server
- Agent
  - "Regular" mode
  - "UKI" mode
- Client
  - GUI based on Bevy
  - CLI based on clap for scripting or optional TUI based on Ratatui

All of these applications are built from the main `sandpolis` crate (except for
the mobile app) with feature flags.

Every crate in the workspace apart from `sandpolis` and `sandpolis-mobile` is a
"layer" that brings some functionality. Layers can depend on each other and some
are optional (controlled via cargo features).

Most layers implement some functionality for all three instance types. For
example, the way to think about the `sandpolis-agent` crate is it "does
something with agents", not that it "implements what an agent does".

## CoLo mode

When a server feature is compiled alongside the client and/or agent and the
binary is run with no subcommand, all instance types start in the same process
and connect to each other automatically over loopback — no `--realm-cert` or
server configuration is needed. This is meant for convenient local testing:
targeting the local instance (e.g. starting a desktop stream) "just works".

## Mobile App

The `sandpolis-mobile` crate wraps the main `sandpolis` crate.

Build instructions for Android:

```sh
cargo ndk -t arm64-v8a -o android/app/src/main/jniLibs build --link-libcxx-shared
cd android && ./gradlew assembleDebug
```

# Roadmap to 1.0

> This project has been in development for a long time and we need to rapidly
> move toward a MVP and then a stable 1.0 release afterwards. This roadmap
> outlines our overall requirements in no particular order.

- "Away" mode where monitoring becomes more strict
  - For example, a SSH login when away is highly suspicious and must be notified
    immediately
- Zooming in on a node enters another level of depth where all other nodes
  disappear. Now shows more detailed operations.
- `DatabaseLayer`, `NetworkLayer`, `RealmLayer` should not be layers anymore?
  Layers vs subsystems? Layers are just UI?
- On desktop, probe, and shell layers: servers are present in the graph (so we
  have links), but they are not interactable. When the server layer is active,
  only servers are shown and they become interactable. Clients are only present
  in the graph when the client layer is active (servers are also present, but
  not interactable).
- Upgrade to bevy 0.19
  - Reimplement all scenes using the new bsn! macro
    (https://bevy.org/news/bevy-0-19/#next-generation-scenes)
    - The pattern and shared scene widgets (`text_line`, `bound_text`, `button`,
      `text_input`, `modal_scrim_scene`) live in
      `sandpolis-client/src/gui/ui/scene.rs` (theme colors via captured `{...}`
      exprs + `Themed*` markers, `template_value` for runtime components,
      `on(..)` observers, dynamic `Vec<impl Scene>` child lists,
      `apply_scene(bsn! { Children [..] })` for building into an existing
      entity).
    - Migrated: help modal, about panel, add-agent modal, theme picker,
      health + inventory controller bodies.
    - Intentionally imperative (bsn! wouldn't simplify them):
      - login modal: tree branches on the login phase and is rebuilt per phase.
      - layer/node pickers: rows are rebuilt on state change into a container
        entity; the static shell alone isn't worth splitting off.
      - layer toolbar: per-button closure/gate capture is the substance.
      - minimap: already declarative via `children![]`; dots rebuild per frame.
      - node/edge graph: rapier physics components lack `Default`, so bsn!
        would need `template_value` wrapping everywhere and read worse.
      - shell/desktop/probe controller bodies: dominated by resource-mutating
        observer closures (and per-device iteration in probe); revisit later.
    - `spawn_floating_panel` (`ui/panel.rs`) intentionally stays imperative: it
      returns child entity ids synchronously, which doesn't fit `spawn_scene`.

## `sandpolis-tunnel`

- Application-level tunnel (traffic to client port gets tunneled to port on
  device in agent/server's network)
  - Implement as stream

## `sandpolis-agent`

- Merge `sandpolis-deploy` crate into `sandpolis-agent`
  - The idea is you can install the agent via SSH or via a local executable
  - Drop outdated code that's no longer useful like the Java/protobuf stuff
  - Drop the embedded config - we're moving towards all configuration happening
    via CLI flags
  - Gate appropriately - systemd features are only needed by the agent, UI
    features are only needed by the client, and SSH features are only needed by
    the server.
  - Scope: just build out the framework, we'll provide the actual prebuilt agent
    binaries for install later

## `sandpolis-account`

- Allow CRUD operations on account objects
- Analyze attack surface
- Compromise tracing:
  - Suppose any entity in the network is compromised, what others could be
    affected?
  - Assign a weight on how bad a compromise of an entity would be

## `sandpolis-snapshot`

- Use boot agent to create/apply "cold snapshots"
- Store snapshots on server
- Not compatible with regular agents


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fossable/sandpolis](https://github.com/fossable/sandpolis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
