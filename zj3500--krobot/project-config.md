---
trigger: always_on
description: This file is the operating contract for agents working in this repository or using the Krobot CLI. Read it before changing code or calling Kroger APIs.
---

# Krobot agent guide

This file is the operating contract for agents working in this repository or using the Krobot CLI. Read it before changing code or calling Kroger APIs.

## Mission

Krobot helps a user turn grocery intent into a reviewed Kroger pickup cart through Kroger's documented public APIs. Optimize for correctness, clear user control, minimal API usage, and secure authentication.

The intended workflow is:

```text
grocery request
  -> select store
  -> search candidates
  -> resolve ambiguity with the user
  -> present a cart plan
  -> receive approval
  -> add approved UPCs
  -> emit a structured browser handoff
  -> continue in Kroger with computer-use
  -> receive final order approval
  -> place the order and report confirmation
```

Krobot's public-API layer does not place orders. An agent may complete the unsupported pickup steps in Kroger's visible website using computer-use, provided it follows the browser-handoff contract below. Final order submission always requires explicit user approval immediately before the click.

## Agent access boundary

The intended shopping-agent workspace is this Krobot directory, not the user's
home directory or the parent containing other projects. Keep grocery lists, cart
plans, and other task files inside this directory; use their absolute paths when
calling the CLI. Use the checkout-local executable. Do not read or write outside
this workspace, follow links outside it, select external state paths, import
legacy state, or change global commands/settings as part of normal shopping.
Ask the user when the task requires additional access.

This guide is an operating policy, not an enforced sandbox. The agent runner
must enforce filesystem and tool permissions. Merely starting in this directory
or reading AGENTS.md does not restrict a process. Krobot currently does not
install or configure an agent sandbox.

Keep secrets outside the agent-readable workspace in secure OS storage
(currently macOS Keychain). Let Krobot retrieve them internally; never inspect
the vault directly, dump environment variables, extract browser cookies, or
request credentials in chat. Credential setup belongs in a user-controlled
terminal; shopper sign-in/MFA belongs in the user's browser. Do not inject secrets
into the agent's environment. Local state in `.krobot/` contains references and
metadata, not secret values, and should still be treated as private user data.

Folder restrictions alone do not restrict credential-store APIs, network access,
or browser tools. The runner must explicitly permit the capabilities needed for
Krobot's official API requests, native vault access, loopback OAuth callback, and
user-approved browser handoff. Do not grant arbitrary shell/network access on
the assumption that filesystem restrictions protect credentials.

For a hard guarantee that an agent cannot retrieve secrets, run credential-bearing
operations behind a separately enforced boundary with only approved operations
exposed to the agent. The agent must not be able to modify that trusted executable
or its launch configuration, or bypass it with shell access under the credential
owner's identity. That isolated execution mode is not implemented by Krobot;
do not claim that this guide or the current CLI provides it. Developing Krobot
source and operating a protected shopping installation are distinct roles.

## Command interface

Use the single `krobot` command:

Krobot is a native Rust executable. Build from the repository root with
`cargo build --manifest-path src/krobot-cli-rust/Cargo.toml --release --locked`.
The source-build executable is `./src/krobot-cli-rust/target/release/krobot`;
release bundles use `./bin/krobot`. Either can be invoked by absolute path.
The examples below use `krobot` as shorthand for that executable, or a
user-configured PATH entry. Do not assume an older `bin/krobot` was refreshed
by a source build; only release packaging copies the new binary there.

- `krobot credentials ce|prod`: securely configure that profile in macOS Keychain.
- `krobot use ce|prod`: choose the default profile for subsequent commands.
- `krobot --profile ce|prod COMMAND`: choose a profile for one command without changing the default.
- `krobot COMMAND`: run against the current default profile.

Never ask the user to paste a client secret, OAuth token, password, verification code, or cookie into chat.

Use `krobot --help` for the complete CLI contract, `krobot help workflow` for the end-to-end flow, and `krobot COMMAND --help` for command-specific syntax and boundaries.

## Environment separation

Use Certification first for development and non-production verification:

```sh
krobot use ce
krobot COMMAND
```

Use Production only when the user explicitly wants real account or cart effects:

```sh
krobot use prod
krobot COMMAND
```

Agents should prefer explicit `--profile` on every stateful or remote command instead of relying on a mutable default. Humans may prefer `krobot use` for shorter interactive sessions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zj3500/krobot](https://github.com/zj3500/krobot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
