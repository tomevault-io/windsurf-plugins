---
trigger: always_on
description: @knowledge/user-preferences.md
---

# RevitMCPBridge2026 - Project Context for Claude Code

## Knowledge Base Imports (Core - Always Loaded)
@knowledge/_index.md
@knowledge/user-preferences.md
@knowledge/voice-corrections.md
@knowledge/session-handoff.md
@knowledge/error-recovery.md
@knowledge/revit-api-lessons.md

## Knowledge Base Overview
The `knowledge/` folder contains **99 files (~1 MB)** of architectural domain expertise. The `_index.md` file provides a complete listing organized by category.

### Knowledge + MCP Integration
```
Knowledge Base (WHAT)     +     MCP Methods (HOW)     =     Intelligent Revit Automation
- Room sizes                    - createWalls()              - Correctly sized rooms
- Code requirements             - placeDoor()                - Code-compliant design
- Material specs                - createSchedule()           - Proper specifications
- Best practices                - setParameter()             - Professional output
```

### When to Read Knowledge Files

| Task | Read These Files |
|------|------------------|
| **Design a room/space** | room-standards.md, [building-type].md |
| **Place elements** | kitchen-bath-design.md, door-hardware.md |
| **Check code** | code-compliance.md, egress-design.md, accessibility-detailed.md |
| **Florida project** | florida-requirements.md |
| **MEP coordination** | mep-coordination.md, [specific-system].md |
| **Documentation** | cd-standards.md, annotation-standards.md |
| **Materials/specs** | material-selection.md, specifications.md |
| **Structural** | structural-basics.md, foundation-types.md |

### Knowledge File Categories (99 files)
| Category | Count | Key Files |
|----------|-------|-----------|
| Building Types | 17 | single-family-residential, multi-family-design, office-design, healthcare-design |
| Structural/Envelope | 12 | wall-assemblies, roof-assemblies, foundation-types, exterior-envelope |
| MEP Systems | 10 | hvac-systems, electrical-systems, plumbing-systems, fire-protection |
| Interior/Finishes | 9 | kitchen-bath-design, door-hardware, millwork-standards |
| Codes/Regulatory | 9 | code-compliance, egress-design, accessibility-detailed, florida-requirements |
| Project Delivery | 10 | construction-admin, cost-estimating, specifications |
| Documentation | 7 | cd-standards, annotation-standards, detail-library |
| Revit/Technical | 8 | revit-workflows, batch-operations, error-handling |
| Workflows | 4 | common-workflows, workflows-pdf-to-revit |
| Emerging Tech | 6 | mass-timber, modular-prefab, renewable-energy, resilient-design |

### Workflow: Using Knowledge with MCP

**Example: "Add a master bathroom"**
```
1. Read: knowledge/kitchen-bath-design.md (clearances, fixtures)
2. Read: knowledge/room-standards.md (minimum sizes)
3. Read: knowledge/plumbing-systems.md (fixture requirements)
4. Execute: createWalls() with proper dimensions
5. Execute: placeFamilyInstance() for toilet, vanity, tub
6. Execute: createRoom() and tagRoom()
```

**Example: "Check egress compliance"**
```
1. Read: knowledge/egress-design.md (travel distance, exit width)
2. Read: knowledge/code-compliance.md (occupancy, construction type)
3. Execute: getRooms() to get occupant loads
4. Calculate: required exit capacity
5. Execute: getElements(Doors) to verify exits
```

## Session Start: Standards Detection
At the start of every session involving Revit work:
1. Read the live system state to see what project is open
2. Get project info from Revit (name, number, client)
3. Match against firm profiles in `knowledge/standards/`
4. Load and apply the matching standards profile
5. If unknown project: offer to analyze and create new profile

## Project Overview
This is a **Revit 2026 Add-in** that exposes the Revit API through the Model Context Protocol (MCP), enabling AI-assisted automation of Revit tasks via natural language.

## Key Facts
- **Total Methods**: 705 (25+ categories including intelligence, validation, compliance)
- **Current Progress**: 705/705 methods registered (100%)
- **Language**: C# (.NET Framework 4.8)
- **Target**: Autodesk Revit 2026
- **Architecture**: MCP Server → Named Pipe → Revit API

## Project Structure
```
RevitMCPBridge2026/
├── src/
│   ├── RevitMCPBridge.cs          # Main entry point, command handler
│   ├── MCPServer.cs                # Named pipe server (282 registrations)
│   ├── WallMethods.cs              # 11/11 methods ✅
│   ├── DoorWindowMethods.cs        # 13/13 methods ✅
│   ├── RoomMethods.cs              # 10/10 methods ✅
│   ├── ViewMethods.cs              # 12/12 methods ✅
│   ├── SheetMethods.cs             # 11/11 methods ✅
│   ├── TextTagMethods.cs           # 12/12 methods ✅
│   ├── ScheduleMethods.cs          # 34/34 methods ✅
│   ├── FamilyMethods.cs            # 29/29 methods ✅
│   ├── ParameterMethods.cs         # 29/29 methods ✅
│   ├── StructuralMethods.cs        # 26/26 methods ✅
│   ├── MEPMethods.cs               # 35/35 methods ✅
│   ├── DetailMethods.cs            # 33/33 methods ✅
│   ├── FilterMethods.cs            # 27/27 methods ✅
│   ├── MaterialMethods.cs          # 27/27 methods ✅
│   ├── PhaseMethods.cs             # 24/24 methods ✅
│   ├── WorksetMethods.cs           # 27/27 methods ✅

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WeberG619/RevitMCPBridge2026](https://github.com/WeberG619/RevitMCPBridge2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
