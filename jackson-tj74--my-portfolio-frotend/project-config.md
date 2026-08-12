---
trigger: always_on
description: 1. **❌ USE TERMINAL COMMANDS TO START/STOP/RUN APPLICATIONS**
---

# ⚡ Copilot Assistant Extension v1.2.3 - Log Access & Tool Guidelines

---

# 🚨🚨🚨 STOP - READ THIS FIRST 🚨🚨🚨

## ⛔ ABSOLUTELY FORBIDDEN - INSTANT FAILURE

### **YOU MUST NEVER, UNDER ANY CIRCUMSTANCES:**

1. **❌ USE TERMINAL COMMANDS TO START/STOP/RUN APPLICATIONS**
   - **NO `npm start`** - Use `copilotAssistant.start` instead
   - **NO `flutter run`** - Use `copilotAssistant.start` instead  
   - **NO `go run .`** - Use `copilotAssistant.start` instead
   - **NO `python app.py`** - Use `copilotAssistant.start` instead
   - **NO `node index.js`** - Use `copilotAssistant.start` instead
   - **NO `java -jar`** - Use `copilotAssistant.start` instead
   - **NO `mvn spring-boot:run`** - Use `copilotAssistant.start` instead

2. **❌ USE TERMINAL TO STOP/KILL APPLICATIONS**
   - **NO `pkill`** - Use `copilotAssistant.stop` instead
   - **NO `Stop-Process`** - Use `copilotAssistant.stop` instead
   - **NO `kill`** - Use `copilotAssistant.stop` instead

3. **❌ USE TERMINAL OR GREP TO SEARCH LOGS**
   - **NO `grep`** - Use `copilotAssistant.searchLogs` instead
   - **NO `Select-String`** - Use `copilotAssistant.searchLogs` instead
   - **NO `Get-Content`** - Use `copilotAssistant.searchLogs` instead

### **⚠️ CRITICAL CONSEQUENCE:**
If you use terminal commands for app control, **logs will NOT be captured**. You will be unable to analyze errors, debug issues, or help the user. The entire purpose of this extension is defeated.

### **✅ ALWAYS USE THESE EXTENSION COMMANDS:**
- **Get instances**: `run_vscode_command({ commandId: "copilotAssistant.getInstancesForCopilot" })` (required before start/stop/restart when an instance is expected)
- **Start instance (preferred)**: `run_vscode_command({ commandId: "copilotAssistant.startInstance", args: ["instanceId"] })` ⚠️ **MUST check compilation first**
- **Restart instance (preferred)**: `run_vscode_command({ commandId: "copilotAssistant.restartInstance", args: ["instanceId"] })` ⚠️ **MUST check compilation first**
- **Start app (fallback only)**: `run_vscode_command({ commandId: "copilotAssistant.start" })` (**ONLY** when no matching instance exists; this may prompt for a directory)
- **Stop instance**: `run_vscode_command({ commandId: "copilotAssistant.stopInstance", args: ["instanceId"] })`
- **Search instance logs**: `run_vscode_command({ commandId: "copilotAssistant.searchInstanceLogs", args: ["instanceId", "pattern"] })`

### **🧠 CRITICAL: MULTI-INSTANCE START RULE (ALL apps)**
If Copilot is asked to start a specific app/project (e.g. **"Start flutter app"**, **"Start node app"**, **"Start the backend"**), you MUST:
1) Fetch instances (`copilotAssistant.getInstancesForCopilot`)
2) Read `.copilot-assistant/instances.json`
3) Choose the matching instance by intent:
  - If user names a language: match `language` (e.g., `flutter`, `nodejs`, `go`, `java`, `python`)
  - If user names an app/project: match `name` and/or `projectRoot`
  - If exactly one instance exists: use it
  - If multiple plausible matches exist: ask the user which instance to start
4) Call `copilotAssistant.startInstance` with the instance ID

**NEVER** call `copilotAssistant.start` first if a matching instance exists — that forces the user to pick a directory.

### **🧠 CRITICAL: MULTI-INSTANCE RESTART RULE (ALL apps)**
If Copilot is asked to restart an app/project (e.g. **"Restart the app"**, **"Restart flutter"**, **"Restart the backend"**), you MUST:
1) Fetch instances (`copilotAssistant.getInstancesForCopilot`)
2) Read `.copilot-assistant/instances.json`
3) Choose the matching instance by intent (same matching rules as start)
4) Call `copilotAssistant.restartInstance` with the instance ID

**NEVER** call `copilotAssistant.start` for an existing instance during a restart request — that forces the user to pick a directory again.

**Note:** If an instance has never been started before, `restartInstance` may tell the user to run Start first. In that case, call `copilotAssistant.startInstance` using the same instance ID.

**Required workflow example (Flutter, but same pattern for any app):**
```typescript
run_vscode_command({
  commandId: "copilotAssistant.getInstancesForCopilot",
  name: "Get all instances"
});

read_file({
  filePath: "<workspace>/.copilot-assistant/instances.json",
  startLine: 1,
  endLine: 200
});

// Pick the matching instance:
// - If exactly 1 matching instance: use it
// - If multiple matching instances: ask the user which one
// - If none: fall back to copilotAssistant.start (directory picker)

run_vscode_command({
  commandId: "copilotAssistant.startInstance",
  name: "Start instance",
  args: [instanceId]
});
```

### **🔍 CRITICAL: PRE-START COMPILATION CHECK**
**BEFORE** executing `copilotAssistant.start`, `copilotAssistant.startInstance`, `copilotAssistant.restart`, or `copilotAssistant.restartInstance`, you **MUST**:
```typescript
// Step 1: Check for language server errors (fast check)
get_errors({})

// Step 2: If errors exist, DO NOT START - fix them first

// Step 3: Run actual compilation to catch build-time errors
// Detect language and use appropriate build command:

// Flutter/Dart:
run_in_terminal({
  // Run this in the Flutter project's root (use the instance's projectRoot when using startInstance)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jackson-tj74/My-Portfolio-frotend](https://github.com/Jackson-tj74/My-Portfolio-frotend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
