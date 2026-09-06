---
trigger: always_on
description: generates a random one when the file is missing *or empty* (`[ ! -s ]`, so a
---

# CLAUDE.md

Notes for anyone — human or otherwise — working in this repository.

User-facing documentation lives in [README.md](README.md): every environment
variable, the DKIM/SRS/SASL/rsyslog setup, the security guidance, the health
check and the deployment examples are documented there, and its
[Testing](README.md#testing) section is the source for the commands below.
This file does not repeat any of it — it covers what the README does not: the
real layout, what gates a merge, and the decisions in the code that look like
bugs but are not.

## What this is

One Docker image: a Postfix SMTP relay for other containers to send mail
through, plus optional OpenDKIM signing, optional SRS rewriting, SASL
authentication on both sides and rsyslog forwarding. Everything is configured
through environment variables, which the `run` entrypoint translates into
config files at container start. A second script, `healthcheck`, is what
docker's `HEALTHCHECK` invokes.

Published as `mwader/postfix-relay`. Issue and pull request numbers cited below
are this repository's own tracker; contributor branches live in forks, which is
why merge commits name someone else's namespace.

## Layout

| Path | Role |
| --- | --- |
| `Dockerfile` | Debian base pin (`FROM debian:trixie-<date>-slim`), the apt packages, the conditional `postsrsd` install, the build-time deletion of `/etc/rsyslog.conf` and `/etc/postsrsd.secret`, the default `ENV` block, `COPY run healthcheck /root/`, `VOLUME`, `EXPOSE 25`, `HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 CMD ["/root/healthcheck"]` and `CMD ["/root/run"]`. No `ENTRYPOINT`, no `ARG`. |
| `run` | The entrypoint. Resolves `<NAME>_FILE` secrets, turns `POSTFIX_*`, `POSTFIXMASTER_*`, `POSTMAP_*`, `OPENDKIM_*`, `POSTSRSD_*`, `RSYSLOG_*`, `SASL_Passwds` and `POSTMASTER_ADDRESS` into config, starts the daemons, asks postfix for an SMTP greeting, then runs a `pgrep`-polling supervision loop. Nearly all behaviour lives here. |
| `healthcheck` | `pgrep`s `master`, checks a listening socket for every `inet` service in `postconf -M`, then `rsyslogd` always, `opendkim`/`postsrsd` whenever the environment *or the artefacts start-up left behind* say so, and `saslauthd` when `SASL_Passwds` is set. |
| `pytest.ini` | `addopts = -n auto --dist loadfile --maxprocesses 4` and one registered marker, `smoke`. No `testpaths`, no `filterwarnings`, no `xfail_strict`. |
| `.dockerignore` | Keeps `.git`, `README.md`, `SECURITY.md`, `LICENSE`, `tests`, `pytest.ini`, `CLAUDE.md` and `.claude` out of the build context. |
| `.claude/settings.json` | Registers the `SessionStart` hook below. Nothing else. |
| `.claude/hooks/session-start.sh` | Starts the docker daemon and installs `tests/requirements.txt`, because a Claude Code on the web container has neither and both gates need them. Guarded on `CLAUDE_CODE_REMOTE=true`, so a local checkout is untouched, and best-effort: a failed step explains itself on stderr and the hook still exits 0, so check stderr before believing a build or test failure. |
| `tests/__init__.py` | Empty; makes `tests` a package, which is what lets `conftest.py` name plugins as `tests.fixtures.*` and lets modules do `from tests.helpers import …`. pytest therefore has to be run from the repo root. There is no `tests/fixtures/__init__.py`. |
| `tests/conftest.py` | Registers the four fixture modules as pytest plugins, and defines the failure plumbing: `print_log_on_failure`, an autouse `shared_container_logs` fixture, and a `pytest_runtest_makereport` wrapper hook that stashes the report on the item. |
| `tests/helpers.py` | The shared vocabulary — `poll_until`, `once_across_workers`, `wait_for_smtp`, `send`, `container_exec`, `postconf`, `listening_ports`, `exit_code_within` and the rest. Imported by every test module but `test_sendmail.py` and `test_ruleset.py`, and by three of the four fixture modules. `file_missing` is the one that cannot be spelled with `container_exec`, which fails on a non-zero exit: asking whether a path is absent needs the exit code, not the output. |
| `tests/requirements.txt` | Seven pinned packages: `dkimpy`, `docker`, `pytest`, `pytest-xdist`, `pyyaml`, `requests`, `testcontainers[mailpit]`. `dkimpy` is what satisfies `import dkim`, so grepping module names against this file looks like a miss when it is not; `pyyaml` is there for `test_ruleset.py` alone. |
| `tests/fixtures/shared_network.py` | Session-scoped `shared_network`: a testcontainers `Network()` with a generated, labelled name — not a fixed one, so an interrupted run leaves nothing for the next one to collide with. |
| `tests/fixtures/postfix.py` | Seven fixtures: `postfix_image`, `upgrade_from_image`, `postfix` and `_relay_pool` (session, the last being the per-configuration relay pool); `postfix_factory`, `postfix_shared` and `docker_volume` (function). Every relay gets `POSTFIX_relayhost=mailpit:1025`, set before the caller's env so a test can override it; readiness is an SMTP connection, not a log line. Reads `POSTFIX_RELAY_IMAGE` / `POSTFIX_RELAY_ARCH` / `POSTFIX_RELAY_IMAGE_PUBLISHED`, and the released image out of `tests/upgrade-from.Dockerfile`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wader/postfix-relay](https://github.com/wader/postfix-relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
