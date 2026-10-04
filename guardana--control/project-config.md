---
trigger: always_on
description: Read this before changing anything in this repository. It applies to every
---

# Project law

Read this before changing anything in this repository. It applies to every
contributor. Where it disagrees with an architecture decision record, the
record wins and this file is wrong; fix it in the same change.

## What this project is

Guardana Control is an inline control and evidence layer for AI agents. The
design is to intercept a consequential action before it happens, decide on it
from structured policy, enforce that decision, and append evidence naming the
policy version that produced it. The first protocol target is the Model Context
Protocol (MCP). An experimental MCP gateway decides and enforces calls;
nothing here is a security boundary yet, and the inventory is
[docs/status.md](docs/status.md).

## Invariants

A change may not break any of these. If a change cannot be made without
breaking one, it needs an architecture decision record before it needs code.

1. No language model runs in the default authorization path. A model may
   suggest, summarise or annotate; it never grants authority.
2. Authorization is structured: principal, delegation, action, resource,
   context, effect class and bounds. Keyword or pattern matching alone never
   grants or denies authority.
3. Public contracts are versioned. An unsupported major version fails loudly
   and visibly, never by ignoring what it does not understand.
4. `INDETERMINATE` is not `ALLOW`. No component may collapse it into one.
5. An action with material effect fails closed when the policy or the evidence
   it needs is unavailable, unless an explicit risk setting says otherwise.
6. An approval binds to one exact canonical action digest and expires. It lets
   that action run once, never a similar or later one.
7. Adapters translate. Policy meaning lives in the core and in policy
   providers, never in an adapter.
8. The core imports no vendor SDK, no store, no server and no adapter.
9. Prompt and tool content capture is off by default, and hidden model
   reasoning is never stored at any setting.
10. Every proposed action, decision, approval and result carries immutable
    correlation identifiers.
11. A non-idempotent effect is never retried automatically.
12. Security behaviour needs adversarial tests. A control with only a happy
    path test is untested.

## No false green

A check that could not run reports an explicit unknown or a failure. It never
reports a pass. No linter enforces this, so reviewers look for it on purpose:
an error assigned to `_`, a `return nil` where an unknown belongs, a check that
an allowlist quietly skips, a documented capability that no test exercises, a
test that asserts on the value it just constructed, a test whose assertions
cannot fail, a gate whose command no-ops when the tool is missing. If you
cannot verify something in the change you are making, say so in the pull
request. An honest unknown is cheap; a green that examined nothing costs
whoever trusts it later.

## Commands

- `make bootstrap` checks Go against `go.mod` and every other tool against
  `scripts/tool-versions.env`, installs a missing one where it can, and fails
  on a mismatch instead of carrying on.
- `make quality-quick` runs formatting, `go vet`, the tests and the import
  check. Use it while working.
- `make quality` runs the whole gate. The `Makefile` names every target it
  calls; read the exact set there rather than from a list in prose that can
  drift.

`ci.yml` runs `make quality` itself, which is the point: a green local run and
a green CI run are meant to mean the same thing. The two machines still differ,
so a change is green when both are. The security and workflow-lint workflows do
not call the gate; they run some of the same tools directly, over a wider scope.
A change is not finished until `make quality` is green.

## Layout

- `internal/` holds what is not intentionally public.
- The public Go surface is `pkg/contract`; `pkg/adapter`, `pkg/detector` and
  `pkg/policyprovider` are reserved. Adding to it is a compatibility decision
  and needs its own record.
- Protocol and framework code lives in `adapters/`, demos in `examples/`.
- Contracts live in `api/proto`, generated Go in `api/gen/go`.
- `internal/core`, `internal/policy`, `internal/canon`, `internal/evidence` and
  `pkg/contract` import only packages of this module outside `adapters/`,
  `internal/storage`, `internal/controlapi`, `internal/gateway` and
  `internal/ingest`; packages of `google.golang.org/protobuf`; and the standard
  library packages named in `scripts/lib/dependency-rule.sh`. That list holds
  no network, file, process, system call or randomness package. In those trees
  the functions of allowed packages that read the clock by name (`time.Now`,
  `Since`, `Until`, the timers, `time.Sleep`, `timestamppb.Now`, the context
  deadlines), standard input, the zone database or the system's randomness are
  refused by name; a read no name reveals, such as the local zone behind
  `time.Unix(...).Format`, is not, and the kernel's tests carry it. A package
  in those trees also builds from the same Go files on every platform: a file
  build constraints leave out here, and any source that is not plain Go, is
  refused.
- Adapters may import core; the rule only constrains what core imports. Core

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guardana/control](https://github.com/guardana/control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
