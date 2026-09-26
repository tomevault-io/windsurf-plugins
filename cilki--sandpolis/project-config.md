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
the mobile app) with feature flags. The agent's "UKI" mode is its own feature:
`uki` implies `agent` and builds the boot agent (always-on chainloader UI, cold
snapshot streams). It is a **compile error** to combine `uki` with `server` or
`client`, which means `--all-features` no longer builds — never use it; always
pass an explicit feature list like `--features server,agent,client`.

Every crate in the workspace apart from `sandpolis` and `sandpolis-mobile` is a
_subsystem_ that brings some functionality. Subsystems can depend on each other
and some are optional (controlled via cargo features).

Most subsystems implement functionality for all three instance types. For
example, the way to think about the `sandpolis-server` crate is it "implements
server-related functionality", not that it is itself the server. Therefore, it's
OK for the `sandpolis-client` crate to depend on `sandpolis-server` to get
shared types (just not with the `server` feature enabled).

A subsystem's runtime state lives in one or more _managers_ — `ShellManager`,
`DatabaseManager`, `NetworkManager`, `RealmManager` — which are constructed at
startup and held together in `InstanceState`.

A subsystem may also provide at most one _layer_, which is strictly a GUI
concept: the mode the client's layer picker chooses between, which decides node
visibility, the toolbar, and the node panel body. `LayerName` names it, and
because the mapping is one-to-one, a layer's name is its subsystem's name.
Notifications and services are attributed to a layer so the client can group
them.

#### Instances

An _instance_ is a Sandpolis process running as an **agent**, **server**, or
**client**. One process is exactly one of them, named by its subcommand:

```sh
sandpolis server   # server daemon
sandpolis agent    # agent daemon
sandpolis client   # client, in the foreground
```

#### Strata

Servers exist in one of two _strata_. Every network has **exactly one global
stratum (GS) server** and **any number of local stratum (LS) servers**.

An LS is an edge cache, useful for on-premise installations where it keeps
serving the instances around it even when the link to the GS is down. It
connects to exactly one GS, and never to another LS.

The distinction decides five things:

- **Configuration.** Only the GS reads _realm configs_ (`<realm>.realm.ron`),
  one per realm it serves, which it finds by scanning its `--data` directory; a
  realm exists only because a file declares it, and can never be created at
  runtime. Every other instance — LS servers, agents, clients — is configured by
  CLI flags plus the _realm cert_ (`<realm>.realm.pem`) naming the server it
  trusts.
- **Trust.** The GS holds the realm CA and is the network's single trust root.
  An LS never generates a CA; on first start it enrolls with the GS, which
  issues it a server certificate. The CA's private key never leaves the GS, so
  an LS can verify peers but never issue certificates of its own. Enrollment
  blocks the listener (with backoff) until it succeeds, since the certificate is
  what the listener presents.
- **Ownership.** Every piece of instance data has exactly one owner at a time:
  the server that instance is directly connected to. A server always owns its
  own scope; estate-wide data (users, accounts, realms) is always owned by the
  GS. The GS arbitrates through a persistent grant table: servers _claim_ their
  attached instances (the GS claims locally, an LS over a claim stream), each
  transfer bumps a fencing epoch, and disconnection is **not** a release — an LS
  keeps its scopes, and keeps writing, through a GS outage and across its own
  restarts. Ownership only moves when an instance shows up attached somewhere
  else.
- **Writability.** `RealmDatabase::write(scope)` gates every write on the
  ownership above: the GS holds full authority, an LS holds
  `WriteAuthority::Scoped`. A freshly granted scope is not writable until it has
  been _hydrated_ — fully replicated down from the GS — so its revision counters
  continue where the previous owner left off. The only paths around the gate are
  replication itself (revision-guarded, so an older record never clobbers a
  newer one) and instance-local bookkeeping (see `RealmDatabase::local_write`).
- **Routing.** Streams cross strata. A client addresses an agent by `InstanceId`
  and never learns the topology: an LS advertises its attached instances to the
  GS, and points its own default route at the GS for everything else.

```sh
# The global stratum server. It serves every realm config in its data
# directory, creating ./data/default.realm.ron if it finds none. A blank realm
# config means "generate a CA for me", which is written back into the file on
# first start.
sandpolis server --data ./data

# Every start also mints one realm cert per realm and writes it to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cilki/sandpolis](https://github.com/cilki/sandpolis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
