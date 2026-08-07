---
trigger: always_on
description: Operational guide for running, extending, and automating **Scheduled Command Executor** with an LLM from the command line.
---


# AGENTS.md

Operational guide for running, extending, and automating **Scheduled Command Executor** with an LLM from the command line.

> Latest release: **v2.8** — correct TZ/DST scheduling across machine vs. job TZ (Cronos with UTC base + job TZ), non-blocking scheduler loop, and `validateCron` lowercase alias.

> Target stack: **.NET 9.0**, Windows (service or console), `Cronos` for cron parsing, `HttpListener` for the dashboard/API.

---

## 1) Agent roles (who does what)

Define these “personas” in your Codex setup; pick one per task.

### 🧠 Core Engineer (Scheduler Agent)

* Owns `CommandExecutorService.cs`, `ConcurrencyManager.cs`, `SchedulerOptions.cs`.
* Guarantees: no crashes on bad config; correct TZ/DST math; graceful shutdown; concurrency safety.
* Key rules: never block the loop; log once per invalid cron; disabled jobs do not execute.

### 🖥️ Frontend Engineer (Dashboard Agent)

* Owns `dashboard.html` and Monitoring rendering.
* Guarantees: loads over `http://localhost:5058/`, responsive (no horizontal page scroll), version chip visible, raw JSON toggle works.
* Keep requests via relative `/api/...` or a robust API base.
* Layout rules: keep the wrapper fluid (~95vw), auto-fit KPI cards, and wrap long table cells while leaving IDs/time columns monospace.

### 🔧 SRE/Operator (Ops Agent)

* Owns `appsettings.json`, URL ACL, Windows Service install/upgrade, logs, alerts.
* Guarantees: HTTP prefix is reserved and matches configuration; service can start without admin prompts.

### 📚 Docs/Release (Docs Agent)

* Owns `README.md`, `AGENTS.md`, changelog sections (“What’s new vX.Y”).
* Guarantees: never lose history; always document new options; provide copy-paste commands.

---

## 2) System prompts you can reuse (paste into Codex)

**Core Engineer (Scheduler) – prompt**

```
Act as the Core Engineer for Scheduled Command Executor. Constraints:
- .NET 9.0 BackgroundService on Windows.
- Use Cronos; handle invalid cron without throwing; log once per bad job; skip disabled jobs.
- Compute next‑run with Cronos using UTC base + job TimeZone (DST safe). Implementation note: call `cron.GetNextOccurrence(DateTime.UtcNow, tz)`. Do NOT pass local DateTime to Cronos.
- Concurrency: TryAcquireAsync + Skip (lock) with 0ms duration.
- Per-job timeout kills process tree; treat service shutdown as success/cancel, not a failure.
Task: <describe the exact change here>
Deliver: a full updated C# file or minimal patch + reasoning.
```

**Frontend Engineer (Dashboard) – prompt**

```
Act as the Frontend Engineer. Constraints:
- Single-file dashboard.html (no build step).
- Must work from http://localhost:5058/ and file:// fallback using API_BASE resolution.
- No horizontal page scrolling; keep the wrapper fluid (~95vw); auto-fit KPI cards; wrap long logs/JSON and long table cells, keeping key columns monospace.
- Show version chip from /api/health; Raw JSON toggle.
Task: <UI change>
Deliver: entire updated dashboard.html.
```

**Ops Agent – prompt**

```
Act as SRE/Operator. Goal: make the service reachable at http://localhost:5058/.
- Ensure URL ACL matches the configured prefix, with trailing slash.
- Provide PowerShell commands to add/delete urlacl, and service install commands.
- Verify with curl /api/health.
Task: <deployment target/environment>
```

**Docs Agent – prompt**

```
Act as Docs/Release. Update README.md preserving all history. Add new options and examples. Keep concise copy-paste commands.
Task: bump to version X.Y, summarize changes, add config fields and API examples.
```

---

## 3) Repository map (for orientation)

```
RunCommandsService/
├─ Program.cs                      # Host/DI/config + hot reload
├─ CommandExecutorService.cs       # Scheduler loop + process execution (TZ/DST safe)
├─ ConcurrencyManager.cs           # Keys + parallel run control
├─ SchedulerOptions.cs             # Poll seconds, defaults
├─ Monitoring.cs                   # HttpListener API + serves dashboard.html
├─ dashboard.html                  # Single-file responsive UI
├─ FileLogger.cs                   # Rolling logs
├─ HealthHttpServerService.cs      # (noop placeholder)
├─ appsettings.json                # Configuration (hot-reload)
└─ Logs/                           # Runtime logs
```

---

## 4) Build / Run / Install (CLI)

### Debug run (console)

```powershell
dotnet build -c Debug
dotnet run --project .\RunCommandsService\RunCommandsService.csproj
```

### Reserve URL (run PowerShell as Administrator)

```powershell
# For local debug under your user:
netsh http delete urlacl url=http://localhost:5058/
netsh http add urlacl url=http://localhost:5058/ user="%USERDOMAIN%\%USERNAME%"
```

### Verify API / Dashboard

```powershell
curl http://localhost:5058/api/health
start http://localhost:5058/
```

### Install as a Windows Service (example)

```powershell
# Publish
dotnet publish .\RunCommandsService\RunCommandsService.csproj -c Release -o C:\Apps\RunCommandsService
# Reserve for LocalSystem (if service runs as SYSTEM)
netsh http add urlacl url=http://+:5058/ user="NT AUTHORITY\SYSTEM"
# Install (example using sc.exe)
sc.exe create "ScheduledCommandExecutor" binPath= "C:\Apps\RunCommandsService\RunCommandsService.exe" start= auto

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peopleworks/RunCommandsService](https://github.com/peopleworks/RunCommandsService) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
