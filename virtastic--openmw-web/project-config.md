---
trigger: always_on
description: Working notes for AI agents (and humans) on this repo. The most important thing here is
---

# AGENTS.md

Working notes for AI agents (and humans) on this repo. The most important thing here is
**where work runs**: this project has a dedicated local build server and test server. Use them.
The developer laptop is underpowered for this codebase and locks up under engine builds.

---

## Where things run

| Host | Reached as | Role |
|---|---|---|
| Hypervisor | `ssh proxmox` | Proxmox. Hosts the VMs below. |
| Build server | `ssh $BUILDER` | Jenkins + Docker. All builds and relinks happen here. |
| Test app server | `$TEST_HOST` | Hosts the app for testing. Deploy target. |
| AI VM | `ssh ai-vm` | Local model inference. |

**Addresses are NOT in this file — this repo is public.** The real values live in
`ci/jenkins/config.env` (gitignored; copy `config.env.example` and fill it in), and the
scripts read them from there or from the environment. Host aliases like `proxmox` /
`ai-vm` are whatever you put in your own `~/.ssh/config`.

Sizing that matters for builds: each VM wants ~10 vCPU and enough RAM to hold a WASM link
step (8GB floor, 24GB ceiling with ballooning works), on an NVMe-backed pool.

---

## Rules of engagement

**DO NOT build the engine on the laptop.** A clean WASM engine compile is ~13 minutes and is
deliberately uncached (see `Dockerfile`: "A clean compile every build is slower but deterministic").
It will make the machine unusable. Run it on the build server.

**DO NOT test by running the app locally.** Deploy to the test app server and test there.

**DO NOT push to `ovhcloud` to test.** That branch deploys to *production*
(morrowind.virtastic.app) via the self-hosted runner on the OVH VPS. The local build server
exists precisely so testing never touches production. Note `mp.virtastic.app` is **not** a
thing: the design that wanted it is dead, and the vhost that kept the name alive has been
deleted (see "One origin" below).

**`dev` IS THE INTEGRATION BRANCH, AND IT IS PUBLIC.** `Virtastic/openmw-web` is a **public**
repo, so everything merged to `dev` is published permanently the moment it lands. Read what you
are pushing. `ci/jenkins/config.env` is gitignored and must stay that way — it holds this
network's hosts and key paths.

`dev` is branch-protected: **one approving review**, stale reviews dismissed, no force-push and
no deletion. Admins are not forced through it (`enforce_admins` is off), so the maintainer keeps
a direct push for emergencies — use it as one.

**Contributors open a PR against `dev`.** The maintainer approves and merges. `main` and
`ovhcloud` are NOT touched by this flow: `ovhcloud` deploys production and is a separate,
deliberate promotion.

---

## The loop

**Merge to `dev`, then run one command.** Builds happen when somebody asks for one -- there is
no polling and no webhook, because triggering Jenkins needs a credential this setup does not
hand out (`/git/notifyCommit` answers 401 and the remote build URL 403 with anonymous read
disabled). Asking is a single command, and it runs everything ON THE BUILD SERVER:

```bash
ci/jenkins/release-to-test.sh            # both images
ci/jenkins/release-to-test.sh engine     # just the WASM engine  (~13 min)
ci/jenkins/release-to-test.sh server     # just the gateway + sim peer
```

It fetches `origin/dev`, restages the build inputs git cannot carry, builds, and deploys --
stopping at the first failure. The deploy runs the contract gate and fails on any miss.

**CLIENT LUA IS BAKED INTO THE ENGINE.** `fsroot/resources/vfs/scripts/mp/*.lua` is packed into
`openmw.data`, so the browser scenario suite runs whatever was in the engine at BUILD time, not
what is on disk. Editing a client script and re-running `mp-harness.mjs` tests the OLD code and
passes, which is worse than not running it. Rebuild the engine (~13 min) before believing a
scenario result about a client-side change. Cheap check:

```bash
grep -c <your-new-symbol> play/openmw.data    # 0 means the harness cannot see your change
```

**RUN IT SO IT OUTLIVES ITS CALLER.** It drives the build server over a single ssh session, so if
the local process dies the ssh dies with it and the remote run stops WHEREVER IT HAD GOT TO. That
is not a visible failure: a run killed between the two deploys shipped the server and not the
engine, every line it had printed said success, and the site kept serving the previous client
while the server ran new code. Nothing in the logs says so -- the tell is the engine hash in
`/index.html` not moving, and the container's age not resetting. Foreground it, or use a job
runner that keeps it alive; `nohup cmd &` from a shell that then exits is exactly the trap.

Confirm a deploy by what CHANGED, never by the exit code:

```bash
curl -sk https://<origin>/index.html | grep -o '__ENGINE_VER = "[^"]*"'   # must differ
ssh <test-host> 'docker ps --format "{{.Names}} {{.RunningFor}}"'         # must have reset
```

| Job | Time | What it does |
|---|---|---|
| server | ~1 min, longer if `openmw/` changed | gateway + sim peer -> deploy -> contract gate |
| engine | ~13 min | WASM engine + statics -> deploy -> contract gate |

Nothing compiles on the laptop. A clean engine build is ~13 minutes and will lock that machine
up; `release-to-test.sh` only ever drives the build server over ssh.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Virtastic/openmw-web](https://github.com/Virtastic/openmw-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
