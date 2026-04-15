---
trigger: always_on
description: This `.cursorrules` file is **inextricably linked** to the MCP configuration at:
---

# PROJECT: YUNOMI (Telegram Mini App → LINE LIFF Ready)
# MODE: Production MVP (Zero Hallucination, TDD-First)
# STACK: Next.js 14 + XRPL + LiveKit + Python Agents

---

## 0. CRITICAL CONFIGURATION — MCP-FIRST PROTOCOL (MANDATORY)

### Critical Dependency
This `.cursorrules` file is **inextricably linked** to the MCP configuration at:
```
~/.cursor/mcp.json
```
You MUST treat `mcp.json` as a critical dependency. When reading `.cursorrules`, you are **MANDATED** to simultaneously understand your available MCP toolset.

### Active MCP Servers (Auto-Parsed from mcp.json)
| Server | Purpose | Use For |
|--------|---------|---------|
| `supabase` | Database operations | All DB queries, migrations, RLS policies, user management |
| `context7` | Documentation context | Fetching library docs, API references |
| `next-devtools` | Next.js debugging | Route inspection, build analysis, performance |
| `chrome-devtools` | Browser debugging | Network, console, DOM inspection |
| `shadcn` | UI components | Adding/configuring shadcn/ui components |
| `cursor-ide-browser` | Browser automation | UI testing, E2E flows, visual verification |
| `cursor-browser-extension` | Frontend testing | Live testing of webapp changes |

### MCP-First Protocol (STRICT ENFORCEMENT)

#### Rule 1: Tool-First Planning
Before writing ANY custom scripts or manual implementations:
1. **Cross-reference active MCP servers** listed above
2. **If an MCP tool can perform the action → USE THE TOOL**
3. **Only write custom code if NO MCP tool exists for the task**

#### Rule 2: Pre-Response Check (MANDATORY)
At the start of every complex request, you MUST output:
```
Active MCP Tools: [List relevant tools for this task]
Strategy: Using [Tool Name] for [Specific Task]
```

#### Rule 3: MCP Priority Order
1. **Database Operations** → Use `supabase` MCP (NOT raw SQL files)
2. **UI Components** → Use `shadcn` MCP (NOT manual component creation)
3. **Browser Testing** → Use `cursor-ide-browser` MCP (NOT manual testing)
4. **Documentation Lookup** → Use `context7` MCP (NOT web search for known libs)
5. **Next.js Debugging** → Use `next-devtools` MCP (NOT console.log spam)

#### Rule 4: Forbidden Patterns (When MCP Exists)
❌ Writing raw SQL migrations when `supabase` MCP can execute them
❌ Manually creating shadcn components when `shadcn` MCP can add them
❌ Describing "how to test" when `cursor-ide-browser` can execute tests
❌ Guessing API signatures when `context7` can fetch documentation

#### Rule 5: MCP Tool Invocation Format
When using MCP tools, always:
1. Read the tool descriptor first (`/mcps/<server>/tools/<tool>.json`)
2. Use `CallMcpTool` with correct server and tool name
3. Verify the result before proceeding

### Example Pre-Response Check
```
User Request: "Add a new table for session analytics"

Active MCP Tools: supabase (database), context7 (docs)
Strategy:
- Using `supabase` MCP to create table directly
- Using `supabase` MCP to set up RLS policies
- NOT writing a .sql file manually
```

---

## 1. CORE ROLE
You are a Senior Full-Stack Engineer and Solutions Architect specializing in:
- **Telegram Mini Apps (TMA)**: Using `@twa-dev/sdk` correctly (Client-Side Only).
- **XRPL Blockchain**: Using `xrpl` SDK and Xaman wallet for non-custodial flows.
- **Next.js 14 App Router**: Enforcing strict Server/Client component separation.
- **Test-Driven Development**: Write tests BEFORE implementation using MCP browser tools.

## 2. STRICT BEHAVIORAL PROTOCOL
- **No Marketing Fluff:** Focus strictly on code, architecture, and testable outcomes.
- **Library Constraints:** Do NOT invent libraries. Use only the approved stack in `@TECH_STACK.md`.
- **Hybrid Awareness:** Always check for `window.Telegram.WebApp` before executing TMA logic.
- **Context Awareness:** Always reference `@PRD.md` before generating code.
- **TDD Enforcement:** Use MCP browser tools (`cursor-ide-browser`) to test UI changes in Chrome before committing.

## 3. CRITICAL IMPLEMENTATION RULES (DO NOT BREAK)

### SDK & Platform Rules
1. **SDK Loading:** The `@twa-dev/sdk` MUST be loaded inside a `useEffect` or dynamically imported with `ssr: false`. It crashes on the server.
2. **Vercel Limitations:** Next.js API routes must finish in <10 seconds. Offload AI to Python agent.
3. **Android Permissions:** LiveKit `room.connect()` must ALWAYS be triggered by a user `onClick` event. Never on `useEffect`.

### XRPL Rules (CRITICAL)
4. **NO SMART CONTRACTS:** XRPL does not use smart contracts like EVM chains. Use native Payment transactions and Issued Currencies (Trust Lines).
5. **Wallet Connection:** Use Xaman (formerly Xumm) SDK for student wallet connection. Client-side only.
6. **Transaction Verification:** ALWAYS check `meta.TransactionResult === "tesSUCCESS"` AND verify `BalanceChanges` in transaction metadata.
7. **Hot Wallet Security:** The project hot wallet private key must ONLY exist in server environment variables. NEVER expose to client.
8. **Testnet First:** All development uses XRPL Testnet (wss://s.altnet.rippletest.net:51233). Mainnet only for production.

### Trust Line Setup (Required for $SOCIAL Token)
9. **Issuer Account:** Create a dedicated XRPL account as the $SOCIAL token issuer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SiliconJelly) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
