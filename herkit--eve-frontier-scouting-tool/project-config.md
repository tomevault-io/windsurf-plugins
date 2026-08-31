---
trigger: always_on
description: A specialized tool for EVE Frontier scout route data that:
---

# Architecture

Server Side: node.js
Client Side: react

# What to build

A specialized tool for EVE Frontier scout route data that:
1. Takes two text inputs containing EVE Frontier system links from scout/mapping tools
2. Extracts all system links in the format `<a href="showinfo:5//XXXXXX">SystemName</a>`
3. Finds the intersection - systems that appear in BOTH inputs
4. Displays results with system names and IDs
5. Exports common systems back into the original format, batched by max 3900 characters per batch

## Key Features

- Extract EVE Frontier showinfo links from both inputs
- Match systems based on their unique system ID (not name)
- Display statistics: total links in each input, common links found
- Show all extracted systems with expandable details
- Export formatted batches with copy-to-clipboard functionality
- Each batch respects 3900 character limit for compatibility with other tools
- Links formatted as: `<a href="showinfo:5//ID">Name</a>→ <a href="showinfo:5//ID">Name</a>→ ...`

## Technical Details

### Link Format
- EVE Frontier uses `showinfo:5//XXXXXX` where XXXXXX is the system ID
- System IDs are unique identifiers (e.g., 30015856)
- System names are NOT unique (can have duplicates)
- Always match/compare by ID, not name

### Export Requirements
- Max 3900 characters per batch in the game client (with hidden markup)
- Game client adds exactly 82 characters of overhead per link (color codes, formatting, etc)
- Separators `→ ` do NOT get overhead - counted at face value (2 chars)
- Tool calculates precise in-game character count: `(linkCount × 82) + rawChars`
- Or more explicitly: `sum(linkLength + 82 for each link) + (separatorCount × 2)`
- Links joined with `→ ` (arrow + space = 2 chars)
- Format: `<a href="showinfo:5//ID">Name</a>→ <a href="showinfo:5//ID">Name</a>`
- Must maintain exact format for compatibility with scout tools
- UI displays both raw character count and expected in-game character count

### Code Organization
- Backend: `server/index.js` - All API endpoints and business logic
- Frontend: `client/src/App.js` - Main React component
- Styling: `client/src/App.css` - All UI styles

## Future Enhancements

Phase 2 will add intelligent route optimization:
- Extract map data from EVE Frontier (system positions, gate connections, distances)
- Build a graph database of the system network (nodes=systems, edges=gates)
- Implement pathfinding algorithms (Dijkstra's/A*) to calculate optimal routes
- Sort intersected systems by optimal path distance/order
- Display route information similar to input format with distances
- Generate optimized batches that follow efficient travel paths
- Show alternative routes and path comparisons

# Example inputs:

Input one:
```
Y:S638 → J:23O1
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30015856">Y:S638</a> 0.00→ <a href="showinfo:5//30015856">Y:S638</a> (4)→ <a href="showinfo:5//30015857">Q:KKR4</a> 8.38→ <a href="showinfo:5//30015849">J:RS4O</a> 7.93→ <a href="showinfo:5//30015861">P:12RK</a> 9.73→ <a href="showinfo:5//30015848">M:13TN</a> 14.45→ <a href="showinfo:5//30015850">G:SE82</a> 27.49→ <a href="showinfo:5//30015902">P:1A91*</a> 15.91→ <a href="showinfo:5//30015904">Z:TIN6</a> 25.89→ <a href="showinfo:5//30015916">D:1408</a> 14.48→ <a href="showinfo:5//30015903">G:142V</a> 10.60→ <a href="showinfo:5//30015910">P:T7K3</a> 14.81→ <a href="showinfo:5//30014291">UB3-Q85</a> 9.67→ <a href="showinfo:5//30014296">ETC-J65*</a> 4.97→ <a href="showinfo:5//30014297">OMR-S75</a> 6.84→ <a href="showinfo:5//30015927">D:10L4</a> 6.65→ <a href="showinfo:5//30014290">O6R-KB5</a> 7.92→ <a href="showinfo:5//30015919">H:16KK</a> 14.96→ <a href="showinfo:5//30015931">Y:15S6</a> 7.21→ <a href="showinfo:5//30015921">Q:2E8R</a> 9.74→ <a href="showinfo:5//30015926">B:VA8E</a> 17.62→ <a href="showinfo:5//30015963">M:L47O</a> 13.14→ <a href="showinfo:5//30015957">J:SK3S</a> 5.00→ <a href="showinfo:5//30015972">M:10KO</a> 38.32→ <a href="showinfo:5//30007294">U:NEAT</a> 38.65→ <a href="showinfo:5//30015859">P:159E</a>

Y:S638 → J:23O1 (Page 2)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [herkit/eve-frontier-scouting-tool](https://github.com/herkit/eve-frontier-scouting-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
