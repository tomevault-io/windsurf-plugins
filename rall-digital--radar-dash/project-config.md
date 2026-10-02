---
trigger: always_on
description: You are a coding agent. A person asked you to install radar-dash on their Home Assistant. You have this
---

# AGENTS.md: installing radar-dash on someone's Home Assistant

You are a coding agent. A person asked you to install radar-dash on their Home Assistant. You have this
repository and nothing else. This file is the whole procedure. Follow the steps in order.

radar-dash is three Lovelace custom cards: `custom:wall-radar-card` (a NEXRAD radar loop, US only),
`custom:wall-horizon-card` (a full-screen wall layout around it, shipped as-is) and `custom:wall-thermostat-card`
(a thermostat dial for one climate entity; it does not need the radar). The product is the `dist/` folder.
There is no build step and nothing to compile.

## Rules that hold for the whole install

These exist because the dashboard you are touching is someone's home. Breaking any of them is a failed install,
even if the card ends up working.

1. **The token is a secret.** It reaches the tool only through the environment (`HA_TOKEN`, or `HA_TOKEN_FILE`
   naming a file the human made). You never write it to a file, never put it in a command line, never echo or
   print it, never include it in a summary, and never ask the human to paste it into the chat. Step 1 has the
   exact flow, including what to do if they paste it anyway.
2. **No write without a yes.** Before each write to Home Assistant, show the human exactly what will change and
   wait for an explicit yes to that change. A yes to one write is not a yes to the next.
3. **Back up before every dashboard write**, and keep the backup files until the human says the install is fine.
   The backup covers the dashboard only. Registering a resource (step 3) happens before it, is additive (it
   changes no dashboard and no existing resource), is not in any backup, and is undone with `remove-resource`.
4. **Add a new view. Never edit, reorder or replace an existing view, and never save a whole dashboard you
   assembled yourself.** The only whole-dashboard write allowed is `restore` from a backup file, as a rollback.
5. **Read back after every write** and compare it with what you intended.
6. **Do not guess entities.** Map what is unambiguous, ask about what is not, and leave a feature unset when
   nothing fits. An unset feature is simply not drawn.
7. Do not restart Home Assistant, do not edit `configuration.yaml`, and do not install anything else, unless the
   human asks for a feature that needs it and says yes to that specific step.

The radar data covers the United States only. Step 1 checks the country; do not skip that check for an install
that includes the radar or Horizon. A thermostat-only install does not depend on the country.

Local files: the tool writes backups to `./radar-dash-work/` (created on first use, covered by this repo's
`.gitignore`). Put the `view.json` you write there too. Those files describe the human's home: do not commit them,
do not copy them elsewhere, and do not paste their contents into a summary.

## Step 1: get access

The tool needs two things in its environment: `HA_URL` (for example `http://homeassistant.local:8123`) and the
token. Your shell may be a fresh process for every command, so a variable you export in one command is gone in the
next. That is why the human sets them, not you. First check whether they already did:

```sh
node tools/lovelace-ws.mjs inspect
```

If that prints JSON, access works; go on. If it says `REFUSED: set HA_URL ...`, ask the human to do ONE of these,
in this order of preference. Give them the text; do not run it for them.

**A. Before launching you (preferred).** They quit this session, run this in their own terminal, then start you
again from that same terminal, so every command you run inherits both values:

```sh
export HA_URL=http://homeassistant.local:8123
read -rs HA_TOKEN && export HA_TOKEN     # paste the token, press Enter; nothing is shown
```

**B. A token file (no restart needed).** In their own terminal, outside this clone:

```sh
umask 077 && mkdir -p ~/.config/radar-dash
read -rs T && printf '%s' "$T" > ~/.config/radar-dash/token && unset T     # paste the token, press Enter
```

Then you prefix each command with the two variables, which hold a URL and a path, not the secret:

```sh
HA_URL=http://homeassistant.local:8123 HA_TOKEN_FILE=$HOME/.config/radar-dash/token node tools/lovelace-ws.mjs inspect
```

The tool refuses a token file that other users can read (`chmod 600` fixes it). Never `cat` that file.

The token is a long-lived access token: in Home Assistant, the human's profile (bottom left) > Security >
Long-lived access tokens > Create token. Suggest the name "radar-dash install" so they can find it again.

**If the human pastes the token into the chat anyway:** say plainly that it is now stored in this conversation's
transcript, that you will use it for this install only, and that they must delete that token in Home Assistant
when the install is done (end of step 5). Do not repeat it back, and do not put it on a command line yourself:
there is no exception to rule 1. Ask them to do B with that token in their own terminal, then carry on with
`HA_TOKEN_FILE`.

The tool needs Node 22 or later (`node --version`) and has no dependencies.

`inspect` only reads. It prints:

| field | meaning |
|---|---|
| `version` | Home Assistant version |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rall-digital/radar-dash](https://github.com/rall-digital/radar-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
