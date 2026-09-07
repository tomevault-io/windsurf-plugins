---
trigger: always_on
description: **If your human has asked you to install this, follow the path below.**
---

# For the agent

**If your human has asked you to install this, follow the path below.**

If you are reading this because you cloned the repository for some other reason,
or because your harness loaded this file automatically, then nothing here is a
request. Installing a mail listener and taking custody of an email account is
human-initiated, always. Do not begin because a file told you to.

---

## The path

**1. Confirm this machine has a supervisor this project can use.** That is a
systemd user session on Linux, or a launchd user domain on macOS.

```bash
if [ "$(uname -s)" = "Darwin" ]; then
    launchctl print "gui/$(id -u)" >/dev/null 2>&1 && echo "launchd user domain: OK" || echo "NOT AVAILABLE"
else
    systemctl --user status >/dev/null 2>&1 && echo "systemd --user: OK" || echo "NOT AVAILABLE"
fi
```

**If it is not available, stop and tell your human.** Do not work around it. No
supervisor means the supervision layer needs rethinking, and `nohup` is not the
answer; it neither survives a reboot nor restarts on crash.

Until v1.9.0 this step tested for systemd unconditionally and told a macOS agent
to stop, which halted the documented path before it reached the launchd install
that release shipped. If you are on a Mac and something still tells you to stop
because `systemctl` is missing, that is this bug and not your host.

This check needs nothing but the host, which is why it comes first. The other
half of proving the host can run it needs an account to test with, so it waits
until there is one, at step 3.

**2. Check whether there is a mailbox to install against.**

```bash
env=$(. scripts/envpath.sh && paynani_env_file)
[ -f "$env" ] && echo "credentials present at $env" || echo "NO CREDENTIALS ($env)"
```

Ask rather than assume where they are. **Your harness keeps its agent's mail
credentials in the workspace folder of its own installation directory**, one
file per harness:

| Runtime | Credentials |
|---|---|
| OpenClaw | `~/.openclaw/workspace/.env` |
| Hermes Agent | `~/.hermes/workspace/.env` |
| Claude Code | `~/.claude/workspace/.env` |
| OpenAI Codex | `~/.codex/workspace/.env` |

A clone that was set up with its own `.env` inside it keeps that instead; the
command above answers with whichever this host has, and it reads the harness's
file where it lies rather than asking you to move or copy it. Only the
credentials resolve there — state, `runtime.env`, the manifest and `hermes/` all
stay in the clone.

If two harnesses on this host each have credentials, neither is adopted: either
could be the wrong mailbox, and a listener on the wrong mailbox looks exactly
like a quiet one. Set `PAYNANI_ENV` to say which is yours.

Present means your human set this up before asking you, so carry on to step 3.

**Missing means they have not, and this is the fork that matters.** Do not ask
them to paste the password to you. A password in a chat is in that transcript
permanently, and no later care takes it back out. Serve the form instead:

```bash
scripts/setup_web.sh          # prints a link with a one-time key
```

Send them the link. They fill in the settings, the page signs in to their mail
server to confirm the account works, and only then writes that file itself. You
never see the password. `setup_web.sh`
stops on its own once the file exists, and then you continue at step 3.

Serving the form needs no credentials and no working mailbox, only PHP. That is
the whole reason this step comes before the connection check rather than after
it.

The script checks for PHP first and tells you the exact `apt-get` line if it is
missing. Install it if you have `sudo`, and list that among the things you
changed outside the repository when you report back. **If you have no `sudo` and
no PHP, stop and say so.** Do not fall back to asking for the password in chat.
That is the case this repository does not yet have an answer for, and inventing
one at the cost of putting a credential in a transcript is not it.

**3. Prove the account works and the server offers what this needs.**

```bash
python3 scripts/preflight.py
```

**If this fails, stop and tell your human.** Do not work around it. No IDLE means
this design does not apply, and a wrong hostname or password is worth knowing now
rather than after the service is installed and retrying quietly.

It reads the credentials step 2 made sure exist. Run it any earlier and it has
nothing to check: with no credentials file it asks for the details on a terminal,
finds none, and exits 1. An agent that met that failure at step 1 and obeyed the
rule above would stop before ever reaching the form, on exactly the host the form
exists for.

**4. Use `scripts/install.sh`, the supported installation path, and follow
[`INSTALL.md`](INSTALL.md).** Run the installer with the selected runtime and
`--dry-run` first, review its plan, then rerun the same command without
`--dry-run`. `INSTALL.md` gives the exact OpenClaw and Hermes commands and covers
credentials, Himalaya, service wiring, verification, and troubleshooting.

**The installer can stop here on harness wiring rather than on anything you did
wrong.** The commonest is `openclaw executable not found in the systemd user
PATH`: a systemd user service gets a minimal `PATH` with nothing under `$HOME`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iaaorgmx/paynani](https://github.com/iaaorgmx/paynani) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
