---
trigger: always_on
description: Diagnose and safely repair Codex Desktop on Windows when openai-bundled plugins such as chrome@openai-bundled and computer-use@openai-bundled are missing, half-cached, installed but unusable, or report native pipe / node_repl / browser-client.mjs problems. Use for Windows Store/MSIX Codex bundled plugin source issues, WindowsApps copy failures, Computer Use unavailable, Browser/Chrome plugin unavailable, or requests to restore openai-bundled without resetting user configuration.
---


# Repair Computer Use

## Core Rules

Always start read-only. Explain each command before running it.

Do not change WindowsApps permissions, do not take ownership of WindowsApps, do not delete the whole `.codex` directory, and do not reset user configuration.

Before any repair, back up:

- `%USERPROFILE%\.codex\plugins`
- `%USERPROFILE%\.codex\config.toml`
- `%USERPROFILE%\.codex\codex-global-state.json`
- `%USERPROFILE%\.codex\.codex-global-state.json`

Treat `browser@openai-bundled` as optional unless the user explicitly asks for it. The usual repair target is `chrome@openai-bundled` and `computer-use@openai-bundled`.

## Quick Workflow

1. Locate this skill directory and run the bundled script in inspect mode:

```powershell
$skill = "C:\path\to\repair-computer-use"
powershell -ExecutionPolicy Bypass -File "$skill\scripts\Repair-CodexBundledComputerUse.ps1" -InspectOnly
```

2. If `openai-bundled` is missing, points into WindowsApps, or plugin/cache files are incomplete, run repair mode:

```powershell
powershell -ExecutionPolicy Bypass -File "$skill\scripts\Repair-CodexBundledComputerUse.ps1" -Repair
```

3. To explicitly include the Chrome native host registry/manifest check, add `-CheckChromeNativeHost`. The script runs this check by default unless `-SkipChromeNativeHostCheck` is passed:

```powershell
powershell -ExecutionPolicy Bypass -File "$skill\scripts\Repair-CodexBundledComputerUse.ps1" -InspectOnly -CheckChromeNativeHost
```

4. If the Chrome native host registry or manifest is missing and the user wants Chrome extension communication repaired, use the explicit native-host repair switch:

```powershell
powershell -ExecutionPolicy Bypass -File "$skill\scripts\Repair-CodexBundledComputerUse.ps1" -Repair -RepairChromeNativeHost
```

This backs up the existing native host registry key and `%LOCALAPPDATA%\OpenAI\extension` when present, then uses the Chrome plugin's own `scripts\installManifest.mjs` to write the manifest and HKCU registry value.

When running from Codex, execute this command through host PowerShell / an approved escalated command. Some Codex sandbox contexts can show a virtualized HKCU view; native host repair must be written to the real user registry hive to affect Chrome.

5. If Codex still cannot expose `node_repl`, Computer Use tools, or Chrome browser runtime after restarting Codex, repeat repair with runtime environment overrides:

```powershell
powershell -ExecutionPolicy Bypass -File "$skill\scripts\Repair-CodexBundledComputerUse.ps1" -Repair -SetRuntimeEnv
```

If both runtime relocation and Chrome native host are broken, combine the explicit switches:

```powershell
powershell -ExecutionPolicy Bypass -File "$skill\scripts\Repair-CodexBundledComputerUse.ps1" -Repair -SetRuntimeEnv -RepairChromeNativeHost
```

6. Restart Codex Desktop after `-Repair -SetRuntimeEnv` or `-RepairChromeNativeHost`.

`-SetRuntimeEnv` also copies `codex-command-runner.exe` and `codex-windows-sandbox-setup.exe` next to the relocated `codex.exe`. This matters when logs show `windows sandbox failed: spawn setup refresh` or `codex-windows-sandbox-setup.exe: program not found` after moving Codex runtime binaries out of WindowsApps.

7. If inspect mode reports stale `openai-bundled` version references in `config.toml`, repair them only with the explicit stale-config switch:

```powershell
powershell -ExecutionPolicy Bypass -File "$skill\scripts\Repair-CodexBundledComputerUse.ps1" -Repair -RepairStaleConfigRefs
```

This switch updates stale plugin cache version segments in `%USERPROFILE%\.codex\config.toml` only after the expected cache directory exists. It does not rewrite `.codex-global-state.json`.

8. Verify CLI state:

```powershell
$codex = "$env:LOCALAPPDATA\OpenAI\Codex\bin\codex.exe"
& $codex plugin marketplace list
& $codex plugin list --marketplace openai-bundled
```

Expected target lines:

```text
openai-bundled          C:\Users\<user>\.codex\openai-bundled-fixed
chrome@openai-bundled        installed, enabled
computer-use@openai-bundled  installed, enabled
```

## Chrome Native Host Check

The bundled script calls the Chrome plugin's own checker:

```text
scripts/check-native-host-manifest.js --json
```

This checks the Windows registry key:

```text
HKCU\Software\Google\Chrome\NativeMessagingHosts\com.openai.codexextension
```

and validates:

- the native host manifest exists,
- the manifest name is `com.openai.codexextension`,
- `allowed_origins` includes the configured Codex Chrome extension ID,
- the registry default value points to the checked manifest,
- the manifest's `path` target exists,
- sibling `extension-host-config.json` exists.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gyratingice-blip/codex-windows-bundled-plugin-repair](https://github.com/Gyratingice-blip/codex-windows-bundled-plugin-repair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
