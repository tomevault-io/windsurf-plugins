---
trigger: always_on
description: This is a **static HTML data visualization dashboard** focused on **US organ trafficking investigation**. The project follows a four-part narrative structure: **Global Investigation → Supply/Demand Analysis → Black Market Impact → Evidence-Based Solution**. Uses **Tableau Public embeds** for interactive visualizations with **Bootstrap 5** for responsive UI and **D3.js** for client-side animations.
---

# VABI Project - AI Coding Agent Instructions

## Project Overview

This is a **static HTML data visualization dashboard** focused on **US organ trafficking investigation**. The project follows a four-part narrative structure: **Global Investigation → Supply/Demand Analysis → Black Market Impact → Evidence-Based Solution**. Uses **Tableau Public embeds** for interactive visualizations with **Bootstrap 5** for responsive UI and **D3.js** for client-side animations.

**Tech Stack:** Pure HTML/CSS/JavaScript (no build tools), Bootstrap 5.3.3, Tableau Public API v1, D3.js v7

**Narrative Focus:** Data-driven storytelling revealing why the US has the world's highest organ trafficking rates despite high GDP, and proposing Iran's policy reforms as a proven solution.

## Architecture & File Organization

### Narrative Structure

The project follows a **four-act data story**:

1. **Dashboard 1 (D1\_\*.html): Global Investigation** - "Why the US?"

   - Establishes US as #1 trafficking destination (105,360 ahead of #2)
   - Reveals economic paradox: High GDP but high Gini inequality + poverty headcount
   - Shows correlation between economic indicators and organ removal cases
   - Includes animated D3 time series (D1_chart11_timeseries.html)

2. **Dashboard 2 (D2\_\*.html): US Organ Supply vs Demand** - "The Imbalance"

   - Legal supply constraints (deceased donors, utilization rates)
   - Demand drivers (disease prevalence, waiting-list mortality)
   - The persistent gap that creates black market pressure

3. **Dashboard 3 (D3\_\*.html): Legal vs Illegal Transplants** - "The Black Market Illusion"

   - Cost comparison: $47k illegal vs $277k legal (deceptive)
   - Total 1-year cost: $1.5M illegal vs $895k legal (reality)
   - Survival rates: 30% vs 90% at 1 year
   - Message: Black markets are economically irrational

4. **Dashboard 4 (D4\_\*.html): The Iran Solution** - "Evidence-Based Reform"
   - Iran's transition from living-led to balanced deceased/living model
   - US vs Iran comparison metrics
   - "Do nothing" cost model (treemap)
   - Policy timeline showing what worked
   - Call to action: Should US adopt Iran's reforms?

### File Naming Convention

- `index.html` - Landing page with topic cards and navigation
- `D[topic-number]_chart[chart-number].html` - Dashboard pages (e.g., `D1_chart1.html`, `D1_chart5.html`)
- Topic prefixes: `D1` = Human Trafficking, `D4` = Organ Supply/Demand (inferred from existing files)

### Page Structure Pattern

All dashboard pages (`D*_chart*.html`) follow this consistent structure:

1. **Sticky Navigation Bar** - Brand: "Data Insights", multi-level dropdown menu for topics
2. **Breadcrumb Navigation** - Shows: Home → Topic Category → Current Page
3. **Content Wrapper** - Max-width: 1400px, centered layout
4. **Visualization Container** (`.viz-box`) - Contains Tableau embed with unique ID
5. **Optional D3 Integration** - Prepared scaffold (often commented out) for Tableau-to-D3 data bridging

## Critical UI Patterns

### Multi-Level Dropdown System

The navigation uses **Bootstrap dropdowns with custom CSS hover behavior**. Key classes:

- `.dropdown-submenu` - Container for nested menu items
- `.dropdown-item.has-submenu` - Parent items with right arrow (`›`)
- `.dropstart` - Auto-applied when submenu would extend off-screen (flips to left)

**Implementation Detail:** JavaScript dynamically adds `.show` class on hover and checks viewport bounds to apply `.dropstart` positioning class.

### Tableau Embed Pattern

Every chart page includes this initialization sequence:

```javascript
var divElement = document.getElementById("viz[UNIQUE_ID]");
var vizElement = divElement.getElementsByTagName("object")[0];
// Responsive sizing: height = 50% of width
vizElement.style.height = divElement.offsetWidth * 0.5 + "px";
```

**Key Details:**

- Uses Tableau Public API v1 (`viz_v1.js`)
- Each embed has a unique container ID (format: `viz[timestamp]`)
- Responsive resize handled with `requestAnimationFrame` debouncing
- Standard aspect ratio: 50% (width:height = 2:1)

### D3.js Integration Scaffold

Dashboard pages include boilerplate for **Tableau-to-D3 data synchronization**:

- `onTableauReady()` - Waits for Tableau API initialization
- `fetchData()` - Extracts selected marks or summary data from worksheet
- `render()` - Creates animated D3 bar charts with transitions (600ms enter, 400ms exit)
- Event listeners: `MARKS_SELECTION`, `FILTER_CHANGE` trigger D3 re-renders

**Configuration Constants:** `WORKSHEET_NAME`, `DIM_FIELD`, `MEASURE_FIELD` (set to `null` for auto-detection)

## Styling Conventions

### Color Palette

- **Brand Purple Gradient:** `linear-gradient(135deg, #667eea 0%, #764ba2 100%)`
  - Used in hero sections, topic icons, and accent elements
- **Active State:** `#667eea` (purple-blue)
- **Neutral Backgrounds:** `#f8f9fa` (light gray)
- **Text:** `#333` (headings), `#666` (body/muted)

### Consistent CSS Classes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nathania02) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
