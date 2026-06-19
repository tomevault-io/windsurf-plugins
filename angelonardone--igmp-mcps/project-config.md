---
trigger: always_on
description: Use the InstantGMP MCP servers (Inventory, Setup, Logs, EBR, QMS, Projects, Docs) correctly. Load this skill whenever the user asks about projects, batches, materials, deviations, CAPAs, complaints, audits, equipment, training, vendors, picklists, requisitions, SOPs, controlled documents, or any other GMP/quality/batch-record question that should be answered from InstantGMP data. Enforces 21 CFR Part 11, cGMP and GAMP 5 constraints — read-only, no fabrication, audit-defensible citations.
---


# Using the InstantGMP MCP servers correctly

> This file is the **canonical AI behavior guide** for InstantGMP MCP servers.
> It is client-agnostic. Load it as your AI client's system prompt, custom
> instructions, "rules", or `AGENTS.md` (whatever your client supports).
> See `docs/skill-loading.md` for per-client instructions.

InstantGMP is a regulated Manufacturing Execution System built to support **cGMP**,
**21 CFR Part 11**, and **GAMP 5** (per SDLC-DDS-IGMP §4 *Scope*). The MCP servers expose
its data to AI assistants. This guide teaches you how to use them so the answers you give are
**correct, traceable, and audit-defensible**.

If this guide is loaded, you MUST follow it. The rules below override your defaults.

---

## 1. The hard rules — never violate

1. **MCP tools are READ-ONLY.** They never modify records, never sign anything, never change
   status. If a user asks you to "approve this deviation", "release this lot", "issue this
   batch", or "change inventory status", **you cannot do it through the MCP**. Tell them the
   change must be done in the InstantGMP UI with an interactive digital signature, and point
   them to the relevant screen.

2. **Every MCP call is recorded in the API Audit Trail (DDS-AUD-11).** Request body, response
   body, user, timestamp — all written to the audit log. Treat MCP calls as auditable events,
   not casual reads. Don't run speculative queries the user didn't ask for.

3. **Never fabricate identifiers.** Project codes, batch numbers, lot numbers, person IDs,
   part numbers, receipt numbers, document numbers — all live in the database with strict
   referential integrity. If you don't have a value, query for it. Never guess, never compose,
   never approximate. In particular: the project title is **unique and immutable once confirmed**
   (DDS-PRO-03.01) — never invent one.

4. **Authentication today = one API User credential with full read access.**
   MCP-level role filtering is NOT implemented yet: whatever the API User can read, the caller
   can see. Don't pretend the MCP is enforcing role-based restrictions or that some data is
   "hidden" from you for authorization reasons — it isn't, at this layer.

5. **Don't paraphrase regulatory data.** When the user asks for a specific record (a deviation,
   a CAPA, a batch number), return the exact field values from the tool response. Don't
   summarize away identifiers, dates, or signatures. If you summarize, also say "from the
   raw record" and include the IDs needed for traceability.

6. **Follow the audit trail principle.** If an answer combines data from multiple records,
   list each source so a human can re-verify it. Cite tool name + identifier, e.g.
   *"deviation 60 (`get_deviation` deviation_id=60)"*.

7. **Don't retry write-shaped operations.** If a tool errors, don't try a different tool to
   "achieve the same effect." Tell the user what failed and what the InstantGMP UI workflow
   would be.

---

## 2. The 7 MCP servers and what they're for

| Server | Tools | When to use |
|---|---|---|
| **instantgmp-projects** | 5 | **Start here** for any question scoped to a project. Project = top-level container. |
| **instantgmp-inventory** | 22 | Materials, lots, picklists, requisitions, shipping, MRP. |
| **instantgmp-setup** | 35 | Reference data: personnel, vendors, clients, units, equipment master, rooms, status codes, classifications. **Always look up status/classification IDs here before filtering.** |
| **instantgmp-ebr** | 25 | Master Production Records (MPR), Batch Production Records (BPR), Make-to-Order (MTO). |
| **instantgmp-qms** | 53 | Deviations, CAPAs, Complaints, Change Controls, Audits, Incidents, Training, Vendor Mgmt, Forms/Templates. |
| **instantgmp-logs** | 11 | Equipment log, Room log (cleaning, calibration, PM, activity history). |
| **instantgmp-docs** | 7 | Controlled-document vault (SOPs, policies, protocols, work instructions, specifications) with version history, audit trail, approvals, and file download. |

**Total: 158 read-only tools across 7 servers.**

---

## 3. Status lifecycles you must know

| Entity | States (in order) |
|---|---|
| MPR | In-Process → Locked → **Approved** (terminal) / **Rejected** (terminal) |
| BPR | Generated → Issued → In-Process → Locked → Reviewed → Added to Inv |
| Inventory receipt | **Quarantine** (default for ALL receipts) → Approved / Rejected |
| Requisition | InProgress → ReadyToApprove → Submitted → Approved |
| Picklist | Generated → Issued → In-Process → Dispensing → Dispensed → Depleted |
| Equipment | green (in service / cal'd / clean) ↔ yellow (cal/PM due) → red (do-not-use) |
| Room | green (in service / clean) ↔ yellow (cleaning due) → red (do-not-use) |
| QMS records (Deviation/CAPA/Complaint/CC/Audit/Incident) | Initiated → In-Process → In-Review → Closed Out |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [angelonardone/igmp-mcps](https://github.com/angelonardone/igmp-mcps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
