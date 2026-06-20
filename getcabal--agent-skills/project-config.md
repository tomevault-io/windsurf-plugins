---
trigger: always_on
description: Cabal's MCP server lets you search people and companies, discover warm introduction paths through your team's network, and manage connectors.
---

# Cabal MCP (Secure)

Cabal's MCP server lets you search people and companies, discover warm introduction paths through your team's network, and manage connectors.

- **Endpoint**: `POST /secure_mcp`
- **Protocol**: MCP `2025-03-26` (JSON-RPC 2.0)
- **Auth**: OAuth 2.0 Bearer token
- **Scopes**: `read` (all tools), `write` (required for `add_connectors`)
- **Server**: `cabal-secure-mcp` v0.1.1

---

## PII Minimization

This endpoint is designed for third-party AI clients (Claude Desktop, ChatGPT, etc.). Tool responses minimize personally identifiable information:

- **No LinkedIn URLs** — replaced with opaque tool IDs (e.g., "Person ID (for tool use)")
- **No Cabal profile URLs** — replaced with opaque tool IDs
- **No email addresses** in search results, connector listings, or message recipient lists
- **No per-recipient message telemetry** — only aggregate engagement counts and rates
- **No timestamps** on messages, imports, or sent dates
- **No inferred connection reasons** (e.g., "Worked together at Google")

Tool IDs (UUIDs) are returned so you can chain them between tools. They are opaque identifiers — do not present them to the user as links.

---

## Core Concept: The UUID Chain

Most workflows follow a **search-then-act** pattern. Search tools return UUIDs; connector and intro tools consume them. Never fabricate UUIDs -- always obtain them from a prior tool call.

| ID Type | Produced By | Consumed By |
|---------|------------|-------------|
| Company UUID | `search_companies` | `get_company_connectors`, `get_company_connections`, `get_connector_intros_at_company` |
| Person UUID | `search_people` | `get_person_connectors`, `add_connectors` |
| Connector UUID | `get_company_connectors`, `get_person_connectors` | `get_connector_intros_at_company` |
| Location name | `lookup_locations` | `search_companies`, `search_people` |
| Industry name | `search_companies_industries` | `search_companies` |
| Investor ID (integer) | `search_companies_investors` | `search_companies` |
| Company UUID (for people filters) | `search_people_companies` | `search_people` |
| People list UUID | `search_people_person_lists` | `search_people` |
| Company list UUID | `search_people_company_lists` | `search_people` |

Always use helper tools (`lookup_locations`, `search_companies_industries`, etc.) to resolve valid filter values rather than guessing.

---

## Tool Reference

### Identity

#### `whoami`

Returns the authenticated user's workspace context.

- **Params**: none
- **Returns**: JSON with name, email, title, team name, and team role

---

### Search

#### `search_companies`

Search the global company database by name, domain, industry, location, size, funding, and investors.

- **Params**:
  - `query` (string) -- company name or domain
  - `industries` (string[]) -- use `search_companies_industries` to find valid values
  - `location_names` (string[]) -- use `lookup_locations` to find valid values
  - `sizes` (string[]) -- employee count ranges (e.g. "11-50", "51-200")
  - `latest_funding_stages` (string[]) -- e.g. "Series A", "Series B"
  - `total_funding_raised` (string[]) -- funding amount ranges
  - `founded` (string[]) -- year founded (e.g. "2020")
  - `investors` (integer[]) -- investor IDs from `search_companies_investors`
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: companies with Company ID, domain, headline, industry, location, size, founded, funding
- **Chains to**: `get_company_connectors`, `get_company_connections`, `get_connector_intros_at_company`

#### `search_people`

Search the global people database by name, company, headline, LinkedIn URL, job function, seniority, and location.

- **Params**:
  - `query` (string) -- person name, company name, headline keywords, or LinkedIn URL
  - `functions` (string[]) -- e.g. "Engineering", "Sales", "Marketing", "Finance", "Human Resources", "Operations", "Product", "Design", "Legal"
  - `levels` (string[]) -- e.g. "C-Suite", "VP", "Director", "Manager", "Senior", "Entry"
  - `location_names` (string[]) -- use `lookup_locations` to find valid values
  - `current_company_uuids` (string[]) -- use `search_people_companies` to find valid UUIDs
  - `former_company_uuids` (string[]) -- use `search_people_companies` to find valid UUIDs
  - `profile_list_uuids` (string[]) -- use `search_people_person_lists` to find valid UUIDs
  - `company_list_uuids` (string[]) -- use `search_people_company_lists` to find valid UUIDs
  - `page` (integer, default 1), `per_page` (integer, default 10, max 25)
- **Returns**: people with Person ID, headline, company, location (no LinkedIn URLs)
- **Chains to**: `get_person_connectors`, `add_connectors`

---

### Filter Helpers

These tools resolve human-readable names into the exact values and IDs that search tools accept. Call them before applying filters.

#### `lookup_locations`

Find valid location values for the `location_names` filter on `search_companies` and `search_people`.

- **Params**: `query` (string, **required**) -- e.g. "San Francisco", "New York", "London"
- **Returns**: matching location strings ranked by relevance

#### `search_companies_industries`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getcabal/agent-skills](https://github.com/getcabal/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
