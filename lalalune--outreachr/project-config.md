---
trigger: always_on
description: Outreachr does not bundle a model or proxy a vendor login. Agents are optional: the rest of the desktop app works without either provider. Codex supports the vendor-owned ChatGPT sign-in flow. Claude uses either a founder-owned Anthropic API key or an existing local Claude subscription session when Anthropic has approved the third-party integration and the founder explicitly enables that mode.
---

# Codex and Claude agents

Outreachr does not bundle a model or proxy a vendor login. Agents are optional: the rest of the desktop app works without either provider. Codex supports the vendor-owned ChatGPT sign-in flow. Claude uses either a founder-owned Anthropic API key or an existing local Claude subscription session when Anthropic has approved the third-party integration and the founder explicitly enables that mode.

## Codex

Outreachr detects the packaged or installed Codex executable and starts its local app-server protocol. In **Settings → Agents**, select **Sign in**, complete the official ChatGPT/Codex browser flow, return to Outreachr, and select **Detect** if the state has not refreshed. The Codex process owns its authentication and stores it in the operating-system keyring; Outreachr does not receive or store the ChatGPT credential. See [Codex authentication](https://learn.chatgpt.com/docs/auth) and the [Codex CLI guide](https://learn.chatgpt.com/docs/codex/cli).

The integration sends a bounded prompt and selected local context. Codex app-server starts with plugins, apps, hooks, and web search disabled. Each run explicitly disables inherited MCP entries, uses an empty environment/capability selection, and requests `approvalPolicy: "never"` with a restricted read-only sandbox. Outreachr gives its authenticated loopback MCP endpoint a unique name so an existing stdio configuration cannot collide with it. Before sending the prompt, Outreachr checks the complete runtime MCP inventory against the run's exact tool allowlist and stops if any other tools are exposed. Any unexpected built-in, app, or MCP tool item interrupts the turn. These overrides apply only to Outreachr's subprocess and do not change the user's Codex configuration. Outreachr uses the configured model only when the bundled CLI lists it as available; otherwise it selects that CLI's recommended default model and reasoning effort. This prevents a model configured in a newer Codex installation from breaking the bundled runtime.

## Claude

Outreachr uses the official Claude Agent SDK with the packaged or installed Claude Code executable. Anthropic's current [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview) says third-party subscription authentication requires prior approval. Outreachr therefore leaves subscription access off by default and never enables it merely because a local Claude login is detected. See [Claude Code authentication](https://code.claude.com/docs/en/authentication), [API authentication](https://platform.claude.com/docs/en/manage-claude/authentication), and [Anthropic legal and compliance](https://code.claude.com/docs/en/legal-and-compliance).

### API-key mode

1. Create a founder-owned key in the [Anthropic Console](https://console.anthropic.com/settings/keys). API usage is billed by Anthropic, so this integration is optional.
2. Open **Settings → Agents** and paste the key into **Anthropic API key**. Do not paste a Claude subscription token or setup token.
3. Select **Save encrypted API key**. The write-only field clears after every attempt. The key crosses the typed preload command once, is encrypted in the main process with Electron's operating-system credential facility, and only ciphertext is stored in the local SQLite vault. Bootstrap/status responses never return it, and production diagnostics must never log it.
4. Select **Detect** if Claude does not show **Ready**. Use **Remove stored API key** to delete the local ciphertext. Credentials are single-device; a restored vault still requires the operating-system credential context that encrypted it.

Saving an API key makes API-key mode active. A previously saved key may remain encrypted as a fallback while subscription mode is enabled, but it is removed from the Agent SDK child environment in that mode.

### Anthropic-approved subscription mode

1. Confirm that Anthropic has approved this Outreachr deployment for third-party subscription authentication. Approval for one distributor or deployment may not transfer to a fork.
2. Run `claude auth login --claudeai` in a local terminal and complete the official Claude Code sign-in.
3. In **Settings → Agents**, check the founder attestation and select **Enable subscription access**, then select **Detect**.
4. To stop using the subscription in Outreachr, select **Disable subscription access**. Outreachr does not log out or alter the independent Claude Code session.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lalalune/outreachr](https://github.com/lalalune/outreachr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
