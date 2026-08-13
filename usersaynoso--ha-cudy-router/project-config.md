---
trigger: always_on
description: - Never restart, reboot, or reload Home Assistant unless the user explicitly asks for it in the current task.
---

# Repository Instructions

## General Home Assistant Operating Rules

- Never restart, reboot, or reload Home Assistant unless the user explicitly asks for it in the current task.
- After deploying to Home Assistant via `ssh ha`, confirm the changed runtime files on the remote system by file contents or checksum before reporting deployment success. Directory presence alone is not sufficient.
- For macOS-originated deploys to Home Assistant, prefer `COPYFILE_DISABLE=1` and exclude or delete `._*` AppleDouble files so Finder metadata does not land in `/config/custom_components`.
- Home Assistant OS targets reached via `ssh ha` may not provide `rsync`; prefer a tar-over-SSH copy staged under `/tmp`, then copy into `/config/custom_components` rather than creating temporary or importable directories inside `/config/custom_components`.
- Keep deploy backups outside `/config/custom_components`, for example under `/tmp`, and still verify the remote checksums before reporting success.
- After a Home Assistant restart or reload, validate only against fresh `cudy_router` log lines from the current boot/reload window.

## Router Validation

- For router behavior bugs, prefer validating against a reachable real router when the user provides access. Use emulators only as fallback or comparison tools.
- When a Cudy status page has a detailed variant, prefer the exact detailed endpoint first, typically `?detail=` or `?detail=1`. Do not generalize this to `?details=` unless the real router proves that spelling.
- For writable client controls, verify both the write path and the subsequent parsed state after refresh/restart; HTTP success alone is not proof of correct behavior.
- When LuCI pages expose multiple hidden inputs for the same control, parse the actual state field rather than the UI toggle-control field.
- `device_tracker` entities are persistent selections, not live-session-only objects. Do not delete them just because the client is currently offline.
- Never leave importable backup directories such as `cudy_router.prev`, `cudy_router.bak`, or `cudy_router.new` inside `/config/custom_components` during deploys. Move backups outside that tree or remove them before restart/reload.
- For R700-style multi-WAN routing, do not assume extra WAN interfaces are only `wand`. Probe `wan`, `wanb`, `wanc`, and `wand`, and verify the returned HTML actually references the expected WAN before trusting it.
- Load-balancing parsers must not require bare labels like `WAN3`. Accept interface cells that embed the WAN token inside labels such as `WAN3 (DHCP)` or `WAN1 / PPPoE`.
- For VPN status polling, do not stop at the first non-empty status page. Probe the relevant VPN status pages and merge them because protocol, tunnel IP, and connected-client count may come from different pages.
- VPN client-count parsing should accept labels beyond `Devices` and `Clients`, including `Connected`, `Connected Clients`, `Online Clients`, and `Peers`, and extract the numeric count from the value.
- When WAN rows are missing, do not assume a DHCP-versus-PPPoE protocol difference is the root cause. First verify the endpoint choice and the returned HTML layout against the real router.
- Distinguish entity registry, device registry, and live entity state when debugging Home Assistant entity issues. Registry presence alone is not proof that an entity is currently loaded or working.
- For options-flow bugs, verify both the saved config entry options and the runtime entity results after reload.
- Keep option labels, translations, README wording, and runtime entity-creation behavior aligned. If an entity type is intentionally excluded from an "automatic" option, say so explicitly in the UI copy.
- For entity-creation bugs, add at least one runtime test that exercises the relevant platform setup path such as `async_setup_entry`. Do not rely only on source-string assertions to validate control-flow behavior.
- For router page-shape or endpoint compatibility fixes, add both a parser fixture test and a `collect_router_data` test that asserts the expected endpoint selection and merged output. Do not rely only on source-string assertions for compatibility changes.
- Connected-client sensors/switches and `device_tracker` entities have separate lifecycles. Do not couple tracker cleanup or picker behavior to automatic connected-device cleanup unless the user explicitly requests that behavior.
- When `Automatically add connected devices` is off, explicit tracked-device selection must still remain possible even when no manual connected devices are selected.

## Home Assistant Frontend Work

- For custom panels or frontend styling work, validate against the live Home Assistant UI after deploy, including both light mode and dark mode.
- Wait for async content to finish loading before judging layout or styling.
- Inspect the live rendered DOM and computed styles when debugging frontend rendering issues.
- Do not assume Home Assistant theme CSS variables are available in every custom panel context, especially iframe-backed panels.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usersaynoso/ha-cudy-router](https://github.com/usersaynoso/ha-cudy-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
