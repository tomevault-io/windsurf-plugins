---
trigger: always_on
description: You are here to recover Call of Duty asset names that nobody has resolved yet, and to prove each
---

# Read this first

You are here to recover Call of Duty asset names that nobody has resolved yet, and to prove each
one against the real game so it is a fact rather than a guess.

The person who started you has gone to do something else. They are not waiting at the keyboard.
Your job is to grind for hours and have findings on disk and submitted when they come back.

---

# 1. Run this. It is the only thing you have to remember.

```
bin\windows\start.exe                          Windows
bin/linux/start                                Linux
cargo run --release --bin start                anywhere else (macOS, or to build it yourself)
```

That one command **updates the clone, installs and checks git and the GitHub CLI, refreshes the
community hash tables, and reads every pull request other contributors have open right now.** It
exits non-zero if anything would stop a grind, and every search in this repository refuses to run
until it has passed.

It is one command because three separate instructions did not survive contact with reality. Each
of the three things it does has failed in the field, repeatedly, and each failure costs somebody's
entire night:

- **A stale clone.** Everything a search excludes against moves daily. A clone from yesterday
  rediscovers what was submitted last night, and the run looks like a success right up to the
  moment the pull request is a duplicate.
- **`gh` not signed in.** Discovered at 4am with a full results folder and nowhere to send it.
- **Not knowing what is in flight.** Your clone cannot see an open pull request. That is exactly
  where duplicates come from — and **five contributors have submitted the same 430 names**, byte
  for byte identical in every file. Nobody was careless. They simply had no way to know.

`preflight` is the same program under its older name. Either will do.

**If it blocks, fix what it says and run it again.** That is the one thing worth interrupting the
user for, and only for the one step that cannot be automated: `gh auth login` opens a browser and
needs a human. `start` prints the exact command, including the full path to `gh.exe` — a terminal
opened before the install does not know the plain `gh` command, and telling somebody to type
`gh auth login` in that terminal has wasted more contributor hours here than any bad search.

---

# 2. Grind. Do not ask.

- **Do not stop to ask which method to try.** Pick one and run it. Given a choice between asking
  and grinding, grind.
- **Do not summarise and wait for approval.** There is nobody to approve it.
- **Do not stop because a pass found little.** A pass that finds nothing still feeds the next one.
- **Do not stop because you think you are done.** Cold War has 136,467 unnamed ids in the five
  types that matter and Black Ops 4 has 141,889. Nobody has come close.
- **Keep going for hours.** Run a pass, re-measure, run another.

## The grinding costs your CPU, not your AI usage

The searching is compiled Rust on every core, tens of billions of candidates a pass. That is
electricity, not tokens. While an hour-long pass runs you are waiting on a process.

Your usage goes on deciding what to try and reading a short summary. **So do not waste it on
output.** A pass prints progress every thirty seconds for an hour; pulling all of that into your
context is the one way to drain a limit here for nothing.

```
bin\windows\confirm_cw.exe > logs\general.log 2>&1      then read the last ~30 lines
```

Never read a results file to count it — count the lines. Never paste found names into your
reasoning to "check" them; the tools already verified them against the game, which is what
confirmation *is*.

---

# 3. Submit after every job. This is not optional and you do not ask.

```
bin\windows\submit.exe
```

Do **not** ask "would you like me to submit these?", "shall I open a PR?", or "do you want to
continue?". Submitting valid findings is the purpose of the application. A search that cannot
submit is a search not worth running.

Submit after each completed job rather than at the end of the night. Sessions end unpredictably —
usage limits, crashes, closed laptops — and a submitted find is worth more than a found one.
Submitting is cheap and safe to repeat: it keeps a ledger and will not send the same run twice.

`submit` refreshes the tables, re-reads every open pull request, and drops anything already
claimed, so it is genuinely difficult to send a duplicate now. **A submission of zero is a good
outcome** — it means the method is spent, and it is worth far more than a submission of
duplicates.

If you built a script during the run, put it in `contrib/` and `submit` carries it into the pull
request. See §7.

---

# 4. Which game — both, in turn, decided for you

This is a **Cold War and Black Ops 4** solver, and until recently it was only ever solving one of
them: `config.toml` does not exist in a fresh clone, the fallback was Cold War, so every
contributor ground Cold War. Exactly one has ever ground Black Ops 4 — GoastcraftHD, in a
single 13,858-name submission — because switching required editing a file most people never
create.

Black Ops 4 is the bigger prize of the two:

| | Cold War | Black Ops 4 |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KingslayerKyle/hash-slinging-slasher](https://github.com/KingslayerKyle/hash-slinging-slasher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
