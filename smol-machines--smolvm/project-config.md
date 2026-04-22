---
trigger: always_on
description: A tool to build and run portable, self-contained virtual machines locally. <200ms boot time. No daemon, no Docker.
---

# smolvm — Agent Reference

A tool to build and run portable, self-contained virtual machines locally. <200ms boot time. No daemon, no Docker.

## Quick Reference

```bash
# Ephemeral (cleaned up after exit)
smolvm machine run --net --image alpine -- echo hello
smolvm machine run --net -it --image alpine -- /bin/sh   # interactive shell
smolvm machine run --net --image python:3.12-alpine -- python3 script.py

# Persistent (survives across exec sessions and stop/start)
smolvm machine create --net myvm
smolvm machine start --name myvm
smolvm machine exec --name myvm -- apk add python3   # installs persist
smolvm machine exec --name myvm -- which python3      # still there
smolvm machine exec --name myvm -it -- /bin/sh
smolvm machine stop --name myvm
smolvm machine delete myvm

# Image-based persistent (filesystem changes persist across exec sessions)
smolvm machine create --net --image ubuntu myvm
smolvm machine start --name myvm
smolvm machine exec --name myvm -- apt-get update
smolvm machine exec --name myvm -- apt-get install -y python3
smolvm machine exec --name myvm -- which python3      # still there after exit+re-exec

# SSH agent forwarding (git/ssh without exposing keys)
smolvm machine run --ssh-agent --net --image alpine -- ssh-add -l
smolvm machine create myvm --ssh-agent --net

# Pack into portable executable
smolvm pack create --image python:3.12-alpine -o ./my-python
./my-python run -- python3 -c "print('hello')"

# Create machine from packed artifact (fast start, no pull)
smolvm machine create my-vm --from ./my-python.smolmachine
smolvm machine start --name my-vm
smolvm machine exec --name my-vm -- pip install requests
```

## When to Use What

| Goal | Command |
|------|---------|
| Run a one-off command in isolation | `smolvm machine run --net --image IMAGE -- CMD` |
| Interactive shell | `smolvm machine run --net -it --image IMAGE -- /bin/sh` |
| Persistent dev environment | `machine create` → `machine start` → `machine exec` |
| Ship software as a binary | `smolvm pack create --image IMAGE -o OUTPUT` |
| Fast persistent machine from packed artifact | `machine create NAME --from FILE.smolmachine` |
| Use git/ssh with private keys safely | Add `--ssh-agent` to run or create |
| Minimal VM without image | `smolvm machine run -s Smolfile` (bare VM) |
| Declarative VM config | Create a Smolfile, use `--smolfile`/`-s` flag |

### Persistence Model

- **`machine run`** — ephemeral. All changes are discarded when the command exits.
- **`machine exec`** — persistent. Filesystem changes (package installs, config edits) persist across exec sessions for the same machine, whether bare or image-based. Changes are stored in an overlay on the machine's storage disk.
- **`machine stop` + `start`** — changes persist across restarts. The persistent overlay is remounted preserving previous changes.
- **`pack run`** — ephemeral. Each run starts fresh from the packed image.
- **`pack start` + `exec`** — daemon mode. `/workspace` persists across exec sessions and stop/start. Container overlay resets per exec (package installs don't persist — use `/workspace` for durable data).
- **`machine create --from .smolmachine`** — creates a persistent named machine from a packed artifact. Boots from pre-extracted layers (~250ms, no image pull). Full `machine exec` persistence — package installs, file writes all survive across exec and stop/start.

## CLI Structure

All commands use named flags (no positional args except `machine create NAME` and `machine delete NAME`).

```
smolvm machine run --image IMAGE [-- COMMAND]     # ephemeral
smolvm machine exec --name NAME [-- COMMAND]      # run in existing VM
smolvm machine create NAME [OPTIONS]              # create persistent
smolvm machine create NAME --from FILE.smolmachine  # from packed artifact
smolvm machine start [--name NAME]                # start (default: "default")
smolvm machine stop [--name NAME]                 # stop
smolvm machine delete NAME [-f]                   # delete
smolvm machine status [--name NAME]               # check state
smolvm machine ls [--json]                        # list all
smolvm machine cp SRC DST                         # copy files (host↔VM)
smolvm machine exec --stream --name NAME -- CMD   # streaming output
smolvm machine monitor [--name NAME]              # foreground health + restart

smolvm pack create --image IMAGE -o PATH          # package
smolvm pack create --from-vm NAME -o PATH         # pack from VM snapshot
smolvm pack run [--sidecar PATH] [-- CMD]         # run .smolmachine

smolvm serve start [--listen ADDR:PORT|PATH]      # HTTP API
smolvm config registries edit                     # registry auth
```

## Artifact References

Artifact references follow OCI conventions and support both tags and digests:

```
python-dev                                        # bare name (default registry + latest)
python-dev:v1.0                                   # name + tag
binsquare/custom:v1                               # namespace + name + tag
smolmachines.com/python-dev:latest                # registry + name + tag
smolmachines.com/binsquare/custom:v1              # registry + namespace + name + tag

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smol-machines/smolvm](https://github.com/smol-machines/smolvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
