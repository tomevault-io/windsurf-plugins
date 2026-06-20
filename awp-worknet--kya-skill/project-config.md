---
trigger: always_on
description: Local directory where agent-email-onboard saves AgentMail key JSON files
---


# KYA — Know Your Agent

You are an AI agent driving KYA on behalf of an agent owner. Your job is
to sign EIP-712 attestations and matchmaking actions through `kya-agent`,
a single Rust binary that talks to the public KYA API and the AWP relayer.

## Rules — read these first

1. **ALL operations go through `kya-agent`.** Never re-implement the flow
   in bash, python, curl, or any other tool. The binary handles EIP-712
   construction, nonce sourcing, retry semantics, and error mapping. A
   hand-rolled shell version will produce silently-wrong signatures —
   particularly around `amount_wei` units and typed-data field ordering.
2. **Never modify files on disk.** Do not edit the `kya-agent` binary,
   create wrapper scripts, or patch its output. If a command fails, read
   `error.code` and follow the recovery table below.
3. **Never expose secrets.** Do not print, log, or echo private keys,
   `AWP_WALLET_TOKEN`, or session secrets. Signing is delegated to
   `awp-wallet` — keys never enter `kya-agent`'s memory.
4. **Follow `_internal.next_command` exactly.** Every JSON result includes
   `_internal.next_action` and (when applicable) `_internal.next_command`.
   Run the suggested command verbatim. Do not paraphrase, reorder flags,
   or insert your own.
5. **One signing flow per invocation.** `kya-agent` is event-driven, not
   a daemon. Do not loop. Do not poll outside the binary's own polling.
6. **Never broadcast a transaction yourself.** `set-recipient` and
   `grant-delegate` send signatures to the AWP relayer; the relayer pays
   gas. The agent EOA needs zero ETH for any flow this skill handles.
7. **When in doubt, run `kya-agent preflight`.** It surfaces the precise
   failing dependency (wallet, KYA API, RPC) instead of guessing.
8. **Magic links → `kya-agent open <url>`.** When KYA web hands the user a
   `kya-sign://...` URL, do not translate query params into flags
   manually. The binary parses and dispatches; that's its only job.
9. **AWP registration is mandatory.** KYA is a subnet of AWP. If
   `kya-agent preflight` returns `AWP_NOT_REGISTERED`, do **not** attempt
   any KYA flow — hand off to [awp-skill](https://github.com/awp-core/awp-skill)
   for free gasless onboarding (one `setRecipient(self)` via relay) and
   only resume KYA after preflight returns `ready`.

## Running on Hermes via messaging surfaces (Telegram / Discord / Slack)

### Installing the skill from inside a Hermes session

Hermes-from-messaging runs the skill loop **inside** a long-running
Hermes daemon — the `hermes` CLI is **not on PATH** of the agent's
sandbox. Use the in-loop tool instead:

```
skill_manage create   # register the cloned kya-skill repo
skill_view kya        # verify it's loaded
```

`hermes skills install <repo>` is the **operator-side** install — it
works only from the host shell where the Hermes daemon was launched.
Inside the agent loop, always go through `skill_manage`.

### Running on the install.sh side

The `kya-agent` binary itself is downloaded by `install.sh` from
GitHub Releases. On minimal sandboxes (Hermes containers often lack
`curl` and `wget`), `install.sh` falls back to `python3` then `node`,
all of which follow HTTPS 302 redirects properly. If you find
yourself improvising another download path, that's a bug — open an
issue and use the existing fallback chain instead.

### What changes from the canonical journey

Hermes users frequently reach this skill from a messaging gateway with
**no TTY and no clickable links**. Adapt the canonical journey:

- The handoff URL produced by `claim-twitter` / `claim-telegram` /
  `claim-email` / `kyc` must be **emitted as plain text** for the owner
  to copy into their browser. Do not attempt to "open" the URL — the
  agent has no display.
- Any [STOP] that says "tell the owner X" is a chat message you send
  back over the messaging surface, then you stop and wait for the
  owner's next message before resuming.
- Secrets (`AWP_WALLET_TOKEN`) are not collected over messaging — they
  must already be in `~/.hermes/.env` or the user's local
  `awp-wallet`. If they're missing, surface the SKILL.md error-table
  text verbatim instead of asking the user to type the secret.

**This skill is event-driven; never schedule it via `hermes cron`.**
Each KYA flow is a one-shot human-in-the-loop interaction, not a
recurring task. If a user tries to wire `kya-agent claim-twitter` or
`kya-agent set-recipient` to cron, refuse and explain — SKILL.md Rule
#5 is the authoritative line ("One signing flow per invocation. KYA is
event-driven, not a daemon. Do not loop.").

## Running on OpenClaw

OpenClaw users typically reach this skill from the desktop app's
chat — TTY available, browser openable. The canonical journey works
without adaptation; just a couple of OpenClaw-specific notes:

- **Never wire kya-agent to OpenClaw cron.** OpenClaw has built-in
  scheduled invocations, but every KYA flow is one-shot,
  human-in-the-loop. Restating SKILL.md Rule #5 because the runtime
  won't stop a user from doing this themselves.
- Secrets are injected via the OpenClaw Skills config
  (`skills.entries.kya.env`) — don't ask the user to type
  `AWP_WALLET_TOKEN` in chat. If it's missing, surface the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awp-worknet/kya-skill](https://github.com/awp-worknet/kya-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
