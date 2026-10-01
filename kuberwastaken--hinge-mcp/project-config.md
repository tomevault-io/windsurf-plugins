---
trigger: always_on
description: Use this guide when the user asks to connect Hinge, validate a live account, or resume a task blocked by authentication. Continue the user's original task after setup succeeds. Do not turn ordinary tool discovery into a login request.
---

# Hinge MCP: instructions for agents

Use this guide when the user asks to connect Hinge, validate a live account, or resume a task blocked by authentication. Continue the user's original task after setup succeeds. Do not turn ordinary tool discovery into a login request.

This guide covers setup, every tool, the other MCP surfaces, and repository maintenance. It is also available through the `hinge://setup` resource and the user-invoked `setup_account` MCP prompt. Fetching either only returns instructions: it sends no SMS, reads no inbox, and changes no account state. If the host only supports tools, use the server instructions and authentication tools below.

## 1. Identify what is missing

- If Hinge tools are available, call `hinge_session_status` first. This reveals local presence/expiry, not whether Hinge accepts the session. When `loggedIn` is true, call `hinge_me` before claiming live access works. A successful read is sufficient; do not print the full profile merely to prove it.
- If tools are unavailable, establish the user's MCP host and where the server will run. Use the [README](https://github.com/Kuberwastaken/hinge-mcp#readme) for the actual host configuration. With authorized shell/config access, build with `npm ci` and `npm run build`, add the absolute CLI path, and reconnect the host. Otherwise give the user the specific configuration step. Preserve unrelated configuration.
- Start account setup with `HINGE_MCP_READ_ONLY=1`; login/logout remain available. Do not enable write/raw tools just to test connectivity. Use one process per session file.
- A transport HTTP 401/403 before a tool runs concerns MCP endpoint access. A Hinge authentication error returned by a tool concerns the Hinge session. Fix the correct layer; sending another SMS cannot fix an MCP bearer token or OAuth configuration.

## 2. Prefer an existing session

Ask only for the **path**, never the JSON or token contents: "Do you already have a hinge-mcp session on the machine running this server? Share its path, or we can sign in with your Hinge phone number."

If authorized to inspect local configuration, check whether `HINGE_SESSION_FILE` is set or the default `~/.hinge-mcp/session.json` exists. Check file existence and ownership without displaying its contents. Limit inspection to the configured/default path or a path the user identifies; do not search unrelated folders, browser profiles, device backups, password stores, or other accounts for credentials.

Point `HINGE_SESSION_FILE` at the user's existing hinge-mcp session and restart/reconnect the server. The path must exist on the **server's machine**; a remote host cannot read a laptop path. Use a private directory and restricted permissions. Do not upload a session file into a chat, repository, issue, or third-party file share. If moving between machines is necessary, have the user use their trusted secret-transfer mechanism or log in on the destination instead.

The server validates the session format and loads it itself. Do not hand-construct session JSON or treat a bare token, browser cookie, or arbitrary app export as compatible. If the configured phone conflicts with the saved session, select the correct account/path or remove the conflicting configuration; do not silently overwrite another account's session. Keep an unreadable file intact while resolving its path/permissions.

Call `hinge_session_status`, then `hinge_me`. Reuse a working session. For an expired/rejected session, explain that a fresh login will replace its local state and proceed with the login flow within the user's authorization.

## 3. Obtain credentials through login

Users do not need to find a Hinge API key, Sendbird token, password, or developer account. This server obtains and saves its credentials through its SMS login flow, with an email challenge if Hinge requires one. It does not implement Google/Apple sign-in. If phone login does not work for the existing account, ask the user to resolve the login method in the official Hinge app/support flow; do not create a replacement account or extract app/browser tokens. Hinge advises using the same method used to sign up: [official login help](https://help.hinge.co/hc/en-us/articles/360011195954-Why-Can-t-I-Login).

Ask only for missing information, one step at a time. A useful initial prompt is: "I need to connect your Hinge account to continue. I can reuse an existing session file, or send a login code to your Hinge phone number. Which would you like?" If the user already requested SMS login and supplied the number, proceed without asking again. A request to read a profile alone is not a request to send SMS or reset login state.

1. Obtain the account's phone number in E.164 format, such as `+15555550123`, or reuse the configured number after establishing it is the intended account. Never infer it from unrelated contacts or messages.
2. Call `hinge_login_start({"phoneNumber":"<user's number>"})` once, or omit the argument when the correct number is configured. It sends an SMS and clears previous local login state. Do not call it repeatedly while waiting for a code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kuberwastaken/hinge-mcp](https://github.com/Kuberwastaken/hinge-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
