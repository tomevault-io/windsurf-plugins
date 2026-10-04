---
trigger: always_on
description: > Fulcra - bridging the gap between agents, other agents, and humans. A context lake for all your data.
---

# AGENTS.md - Fulcra

> Fulcra - bridging the gap between agents, other agents, and humans. A context lake for all your data.

## About

[Fulcra](https://fulcradynamics.com/) is a personal data platform that gives humans and their agents a place to collect, store, and share real-world personal data - calendars, location, files, records that people and agents write, and data from phones, wearables, and other connected devices. Hundreds of data sources and types are supported.
Users and agents can also record new data of their own - events, measurements, notes, and progress toward a goal - and define their own data types for it.

Data is primarily collected through the human's phone; the human installs [Context by Fulcra](https://apps.apple.com/us/app/context-by-fulcra-health-hub/id1633037434) and lets the app sync their data to their account.


### Interactive Access to the User's Data
The human user gets to investigate their data interactively using beautiful mobile and [web apps](https://context.fulcradynamics.com/).

### Agentic/Programmatic Access To the User's Data
* [Main Developer Docs](https://docs.fulcradynamics.com/)
* [OpenAPI spec](https://api.fulcradynamics.com/openapi.json)
* [Python client library and CLI](https://fulcradynamics.github.io/fulcra-api-python/) (`pip install fulcra-api`): For an easy way to use the client library. Handles authentication for you. It also includes the `fulcra` CLI, which gives command-line access to the full platform — data queries, the data type catalog, tags, user-defined data types, and file storage. Sub-commands emit JSON lines for piping into tools like `jq`; run `fulcra --help` for the command list.
* [MCP server](https://mcp.fulcradynamics.com): The endpoint to the public MCP server. The server uses Streamable HTTP transport with OAuth2 authorization. Context users can use this server with their own account to securely access their data.
* [MCP server source code](https://github.com/fulcradynamics/fulcra-context-mcp): The open-source repository for the MCP server. Useful for inspecting available tools, running locally, or contributing.

### For Agents and LLMs: Authorization Tips

#### Code-first agents

If you can run shell commands, try the `fulcra` CLI first; it ships as part of the `fulcra-api` client library (`pip install fulcra-api`). Authenticate once with:

```
fulcra auth login
```

This uses the OAuth2 Device Authorization Flow: it prints a URL for the operator (the user) to open in a browser, polls until they approve, and persists credentials (including a refresh token) at `~/.config/fulcra/credentials.json`, so subsequent commands need no re-authentication.

If you can't keep a process alive while the user completes the browser flow, use the split, non-interactive variant:

```
fulcra auth login --get-auth-url          # prints the auth URL and a device code; send the URL to the user
fulcra auth login --device-code <CODE>    # run after the user finishes the browser flow
```

To make direct REST API calls, `fulcra auth print-access-token` prints a bearer token, e.g.:

```
curl --oauth2-bearer "$(fulcra auth print-access-token)" 'https://api.fulcradynamics.com/user/v1alpha1/info'
```

If you're writing Python code, the same flow is available on the `fulcra-api` module. When calling `.authorize()`, the output will include a URL that you can send to the operator; the call polls while the user completes it in a browser. If the call times out, call `authorize()` again to get a new URL.

```
>>> from fulcra_api.core import FulcraAPI
>>> fulcra = FulcraAPI()
>>> fulcra.authorize()

            Use your browser to log in to Fulcra.  If the tab does not open
            automatically, visit this URL to authenticate: https://fulcra.us.auth0.com/activate?user_code=DBNV-DBQV
```

#### Text-first agents

For agents without the ability to run shell commands or Python code, use the [MCP server](https://mcp.fulcradynamics.com). This server includes tools that can access the same data sources that the API can.

The user can either use the public MCP server instance at `https://mcp.fulcradynamics.com`, or run it locally. It is published as the `fulcra-context-mcp` PyPI module.

You can run it locally (stdio transport) with `uvx fulcra-context-mcp@latest`. See the [PyPI page](https://pypi.org/project/fulcra-context-mcp/) for more docs.

**A CLI login is not a hosted MCP login.** The hosted server at `https://mcp.fulcradynamics.com` runs its own OAuth2 service and only accepts access tokens it issued itself through that flow; it does not accept the access token that `fulcra auth login` (or `fulcra auth print-access-token`) writes to `~/.config/fulcra/credentials.json` — presenting that token as a bearer to the hosted server fails with a 401 `invalid_token`. If an agent has already authenticated via the CLI, running the MCP server locally (previous paragraph) picks up those same credentials automatically. To use the hosted server instead, the MCP client must complete its own OAuth2 authorization with `https://mcp.fulcradynamics.com` (a separate browser login), which is independent of any prior CLI login.

#### MCP Client Configuration Examples


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fulcradynamics/fulcra-context-mcp](https://github.com/fulcradynamics/fulcra-context-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
