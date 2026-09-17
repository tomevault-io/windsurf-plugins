---
trigger: always_on
description: End-user documentation lives in [`docs/`](docs/README.md). It is written for the
---

# AGENTS.md — agent layer over `docs/`

End-user documentation lives in [`docs/`](docs/README.md). It is written for the
person who installs a host and deploys apps. This file is for agents changing
the engine. Never duplicate a user procedure here. Link to the baseline page,
then add only what an agent needs: how to verify, what numbers to report,
traps, internals.

`docs/` never links here. A public wiki export of `docs/` stays clean.

The rule throughout: every claim is a measurement, and every measurement names
the stage it belongs to. "Deploy took 100s" is not a result. "npm ci 16.9s,
apt-get 11.9s, build 2.2s, inside a 53.1s critical path" is.

| Topic | Baseline (operator) | This file |
|---|---|---|
| Install, tokens, TLS | [`docs/02-getting-started/`](docs/02-getting-started/install.md) | `--in-container` for tests; installer prints `pae-artisan` |
| MCP | [`docs/04-connecting-your-ai/`](docs/04-connecting-your-ai/your-assistant.md) | CatalogueTest; do not invent client UIs |
| Detection / stacks | [`docs/07-supported-projects/`](docs/07-supported-projects/how-detection-works.md) | §6a Railpack measurements; §11 onboarding an app |
| Deploy failures | [`docs/02-getting-started/what-happens.md`](docs/02-getting-started/what-happens.md) | Explainer rules vs DinD proof |
| Telemetry | [`docs/02-getting-started/what-is-collected.md`](docs/02-getting-started/what-is-collected.md) | Field list when changing `DeployReport` |
| Pipeline speed / caches | (none — operator does not measure this) | §1–§10 below |

> **Adding support for a third-party application?** §11 in this file is the
> playbook: detect without paying for a deploy, when an app needs a manifest,
> the failures that actually come up, and what counts as proof. Sections 1–10
> verify the *pipeline*; §11 onboards an *application*.

---

## 1. Unit tests

```bash
cd core
./vendor/bin/phpunit --testsuite Unit
```

**Current baseline: 2140 tests, 7986 assertions, 2 failures.**

Those 2 failures are pre-existing and unrelated to Deploy:

| Test | Cause |
|---|---|
| `ChangeWebserverSystemTest::test_webserver_script_parse_args_handles_php_argument_order` | `scripts/webserver-parse-args.test.sh` is missing from the repo |
| `UserIpAddressesTest::test_get_bind_ip_addresses_filters_non_local_default_ipv4` | depends on the host's default IPv4 route |

Report them as pre-existing **only after proving it** — `git stash`, re-run,
confirm the same 2 fail on the baseline, `git stash pop`. Never wave a failure
away as "probably pre-existing".

A recipe change must also keep detection stable. The fastest proof is
differential: extract the pre-change classes into a parallel namespace, run both
over the same fixtures, and diff every field of the returned decision.

---

## 2. Deploy tests (DinD)

```bash
php scripts/dind-test/deploy.php <git-url|local-path> [flags]
```

Runs the **real** engine code (`Dind`, `DetectProjectStrategy`,
`FrameworkDockerfile`, `HostCompile`) against a real Docker-in-Docker account.
`TestSystem` only strips `sudo`, neutralises `chown`, and redirects host paths.

### Flags that change what you are measuring

| Flag | Effect | When you need it |
|---|---|---|
| `--real-home` | Account at `/home/<container>` instead of the disk cache dir | **Required** for nginx-static and Nitro-standalone recipes. `HostNodeBuild::isSafeProjectDir()` and `Dind::emptyProjectDir()` accept only `/home/<user>/project`; anywhere else the host compile throws `Refusing host Node build outside ~/project` and that whole code path goes untested |
| `--reuse-container` | Redeploy into the running container | **Required for any warm measurement** — see §4 |
| `--keep-cache` | Keep the host build cache (node_modules/npm/pnpm/yarn/bun) | Measuring host-compile reuse |
| `--name=NAME` | Account name (container is `dind-test-NAME`) | Reusing one prepared `/home` dir for many apps |
| `--timeout=N` | Seconds to wait for the app | Slow builds |

Two traps worth knowing before you stage anything: a **cold run wipes the
account directory**, and a **local path is cloned with git, so it deploys
`HEAD`, not your working tree**. Both are covered, with the rest of the
onboarding loop, in §11.

The tester removes **only its own container**, by name (`docker rm -f
dind-test-NAME`). It used to `docker container prune -f`, which is unscoped and
on a host with real hosting accounts deletes every stopped container — including
the shared cache registry. If you are reading an older checkout, check that line
before running it anywhere that matters.

`--real-home` needs the directory to exist and be yours (creating it under
`/home` needs root, once per account name):

```bash
sudo mkdir -p /home/dind-test-NAME && sudo chown $(id -u):$(id -g) /home/dind-test-NAME
```

---

## 3. What the tester reports, and what to copy into a result

Every run ends with a summary. **Report all of it** — strategy, railpack,
mode, build breakdown, phases:

```
  Strategy:  NestJS (nestjs)
  Railpack:  no
  Mode:      cold (fresh container)
  Build:     9 layer(s), 0 cached, 32.9s spent building
  Time:      103.8s total (create 0.8s, daemon-wait 1.2s, daemon-settle (test-only) 2s,
             seed (test-only) 42s, clone/detect 0.3s, start 50.8s, settle (test-only) 5s,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panelalpha/panelalpha-engine](https://github.com/panelalpha/panelalpha-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
