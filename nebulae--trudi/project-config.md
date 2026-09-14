---
trigger: always_on
description: **Live-monitoring case** — Velociraptor-backed continuous detection on a
---

# Case: DEMO-LIVE

**Live-monitoring case** — Velociraptor-backed continuous detection on a
Dockerized Linux victim. Unlike static-evidence cases, there is no E01 or
memory image; the "evidence" is a stream of events from `Custom.TRUDI.*`
event artifacts running on the client. TRUDI does **not** execute
remediation — the `respond.*` namespace only *recommends* operator-runnable
containment commands (Velociraptor's client build ships without `execve()`,
and keeping execution out of the agent's hands is the safer posture anyway).

---

## Case Metadata

| Field | Value |
|-------|-------|
| **Case ID** | DEMO-LIVE |
| **Case type** | Live monitoring (Velociraptor) |
| **Target host** | `victim` (Docker container, Ubuntu 22.04) |
| **Velociraptor client_id** | `C.e6de3278ad209722` |
| **Demo env path** | `~/trudi/demo/live-monitoring/` |
| **Opened** | 2026-06-05 UTC |
| **Investigator** | trinity@mentalpad.net |

If the client_id changes (after `docker compose down -v`), update it here
and refresh `~/cases/.common/active_case`.

> client_id history: `C.b26bf213f23d389a` → `C.a7e09ef2f7d2932b` →
> `C.c121f49108aac429` → `C.e6de3278ad209722` (re-resolved 2026-06-12 by
> `/trudi-start-watcher`; victim re-enrolled).

---

## Case Question

**Is there anomalous activity on the monitored host that warrants
containment or eradication?**

Use this as the `observation` of the initial `reason.hypothesize` call
on every alert-driven investigation (per DAIR Triage rules in
`~/.claude/CLAUDE.md`).

---

## Workflow

1. **Baseline** — `monitor.baseline_capture(client_id, "DEMO-LIVE")`
   snapshots processes / persistence / network into
   `monitoring/baselines/<client_id>.json`.

2. **Start watcher** — `monitor.start_watcher(client_id, "DEMO-LIVE")`
   renders detector artifacts from the baseline, uploads them to the
   server, registers them on the client event table, and spawns
   `bin/trudi-velo-watcher.py` as a detached sidecar that drains
   `watch_monitoring()` into `monitoring/alerts/<seq>_<detector>.json`.

3. **Poll for alerts** — `/loop 15s /trudi-check-alerts` (in a Claude
   session). Each tick drains new alerts via `monitor.check_alerts`,
   opens (or extends) one investigation per tick with
   `monitor.start_investigation` (id `INV-NNN`), runs ONE
   `reason.hypothesize` → `dair_assess` → priority-tool batch →
   `misc.record_finding` chain over the whole bundle, and for any
   CONFIRMED/LIKELY finding auto-calls `respond.suggest_containment`.
   `monitor.end_investigation` exports
   `reports/<case>_<INV-NNN>.{json,md}` and flips the active trace
   back to the case-wide file.

4. **Respond (recommend only)** — for each CONFIRMED/LIKELY finding,
   `respond.suggest_containment` renders a copyable `manual_command` from
   the detector's recipe with the alert evidence substituted in. The
   `/trudi-check-alerts` skill surfaces these as a numbered `ACT-N` menu
   (description, risk/reversible, fenced command) after findings but before
   `end_investigation`, and `end_investigation` writes them into the report's
   **Recommended Containment Commands (run manually)** section. TRUDI never
   executes them — the operator runs any command out-of-band. The only gate
   on `respond.*` is `live_monitoring_scope`.

---

## Directory Layout

```
~/cases/DEMO-LIVE/
  CLAUDE.md                              this file
  .claude/settings.json                  MCP tool allowlist for this case
  analysis/                              (all trace files flat under here
                                          so the dashboard scan picks them up)
    DEMO-LIVE_trace.json                 case-wide ORCHESTRATION trace
                                         (check_alerts, list_watchers, ack_alert,
                                          next_investigation_id,
                                          start/end_investigation markers)
    DEMO-LIVE_INV-001_trace.json         one trace per investigation, where
    DEMO-LIVE_INV-002_trace.json           an "investigation" = the bundle of
    ...                                    alerts drained in one /loop tick
                                          (kept open across ticks if approvals
                                          are pending)
  reports/
    DEMO-LIVE_INV-001.json               auto-exported from end_investigation
    DEMO-LIVE_INV-001.md
  exports/                               on-demand CSV / JSON dumps
  monitoring/                            (created on first baseline_capture)
    _inv_seq.txt                         per-case monotonic INV counter
    _open_investigation.json             present iff an investigation is open
                                          across /loop ticks (tracks
                                          investigation_id + alert_ids)
    baselines/<client_id>.json           allowlist snapshot
    artifacts/Custom.TRUDI.*.yaml        rendered detector artifacts (audit copy)
    watchers/<client_id>.{pid,log}       sidecar state
    alerts/                              one file per detector event (the payload)
      _seq.txt                           monotonic sequence
      _seq.lock                          fcntl lock around _seq.txt
      <seq>_<detector>.json              alert payload

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nebulae/trudi](https://github.com/nebulae/trudi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
