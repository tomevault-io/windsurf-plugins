---
trigger: always_on
description: Use for Tilde deployment, provisioning, and home-management work. Trigger for Tilde prompt markers such as `/tilde ...` or `$tilde ...`, `~ ...`, messages whose first word is `tilde`, or requests to create, initialize, deploy, update, diagnose managed state, adopt app/config files, customize data-layer policy, or work with Tilde home behavior.
---


# Tilde

Tilde is a standalone agent skill and control plane. The canonical contract is
[Specification](references/specification.md). Read that file before changing behavior or executing nontrivial Tilde
work. The `.agents/specs/tilde.md` path is only a symlink to the same canonical file.

The provisioning model is:

- Desired state comes from the current committed public/private home repositories.
- The only persistent Tilde state file is `~/.local/state/tilde/state.yml`.
- `state.yml` stores repository bindings and the last fully converged commit anchors.
- Planning evaluates current manifests and targeted live facts on every run.
- Every run compares current desired manifests with targeted live checks and idempotent actions.
- Failed, conflicted, or deferred runs do not advance `applied` anchors.
- The host-local lock is process coordination, not cleanup state.

When existing docs, code, habits, or memory conflict with `references/specification.md`, the specification wins.

## Agent Quickstart

When the user invokes a Tilde prompt such as `/tilde <command> [target] [qualifiers...]`:

1. Treat `/tilde`, `$tilde`, and similar Tilde prompt markers as agent prompt contracts. They are not shell commands.
2. Resolve the controller-side runtime entrypoint before shell execution. Use the loaded skill directory's `bin/tilde`;
   if that path is not available, use `~/.agents/skills/tilde/bin/tilde`. Do not rely on bare `tilde` being on `PATH`.
3. Identify the target: current host, the current host name, `host`, `ssh:host`, or an all-caps host group such as
   `ALL`, `HOME`, or `WORK`.
4. Classify the command from the Command Reference below.
5. Map agent-orchestrated commands to the Workflow Matrix before running helpers.
6. For remote targets, run the Remote Freshness Preflight before reading target state, generating plans, or applying
   results.
7. Use the resolved runtime entrypoint for script delivery. Generate plans, run live checks, and apply results on the
   target host, not on the controller.
8. Keep proposal-first behavior for destructive, preference-sensitive, privilege-requiring, or remote mutations.
9. After a successful mutating remote apply, run a cheap final status read on the target before closeout.
10. After the run, summarize successful, changed, deferred, conflicted, and failed work.

If a direct runtime call says a command is agent-orchestrated, stop trying shell variants of that command. Load this
skill, classify the command, and run the appropriate agent workflow.

Do not execute `/tilde ...` or `$tilde ...` in a terminal. `/tilde` is a prompt marker, and `$tilde` may be a Codex skill
trigger or shell variable expansion depending on the environment. Shell execution uses the resolved runtime entrypoint,
written as `"$TILDE"` in examples:

```bash
TILDE=${TILDE:-"$HOME/.agents/skills/tilde/bin/tilde"}
"$TILDE" help
"$TILDE" ssh spinoza
```

If the skill was loaded from another directory, set `TILDE` to that directory's `bin/tilde` instead. If a controller-side
`tilde` command is not found, do not search the filesystem for it; switch to the resolved runtime entrypoint.

For remote targets, the first target-state read must happen on the target through Tilde SSH transport. Do not
inspect the controller's `~/.local/state/tilde/state.yml` to discover a remote host's repository bindings, applied
anchors, level, platform, or bootstrap state. That file belongs only to the controller host.

## Target Grammar

Host-aware prompt commands accept these target forms:

- no target: current host, also called localhost.
- `host`: remote host shorthand for `ssh:host`, except when it names the current host.
- `ssh:host`: explicit remote host target.
- `GROUP`: a bare all-caps host group defined by the active home policy, such as `ALL`, `HOME`, or `WORK`.

If a bare host token equals the current host name, such as `hostname -s`, treat it as the current host and run the local
workflow. Use explicit `ssh:host` only when the user really wants SSH transport to that host, including self-SSH.

The host-aware prompt commands are `deploy`, `update`, `repair`, `upgrade`, `align`, `status`, and `doctor`.
Commands with their own subject syntax, such as `adopt APP_OR_PATH`, `clean SUBJECT`, `organize SUBJECT`, `create`, and
`init`, do not treat a bare argument as a host unless the command's own policy says so.

Bare all-caps targets are not hostnames. Expand them from the active `~/AGENTS.md` home policy before running any remote
work. Group expansion applies only to unprefixed target tokens; `ssh:host` is always an explicit host target. If the
policy does not define the requested group, ask the user for the host list. For explicitly requested configured groups,
report the expanded host list and continue; the prompt itself is consent to run the requested workflow. Ask only when
the group is undefined, ambiguous, unexpectedly expands outside active policy, or a later step requires separate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roktas/tilde](https://github.com/roktas/tilde) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
