---
trigger: always_on
description: **Crispy Print** is a Frappe v15+ custom app that provides a modern print format builder using **Typst CLI** as the backend print engine and **Vue 3** as the frontend visualization layer. It integrates into Frappe Desk as an SPA and offers live preview capabilities for creating print formats with Typst markup.
---

# Crispy Print - AI Coding Instructions

## Project Overview

**Crispy Print** is a Frappe v15+ custom app that provides a modern print format builder using **Typst CLI** as the backend print engine and **Vue 3** as the frontend visualization layer. It integrates into Frappe Desk as an SPA and offers live preview capabilities for creating print formats with Typst markup.

**Core Architecture:**
- **Backend:** Python-based Frappe app (v15+) with minimal backend logic
- **Frontend:** Vue 3 SPA with Vite build system
- **Print Engine:** Typst CLI integration via Web Workers (non-blocking UI)
- **Integration:** Dual-mount architecture (Vite dev mode + Frappe Desk integration)
- **Inspiration:** Draws patterns from Frappe's `Print Format` DocType, `print-format-builder-beta`, and sibling `apps/crispy` implementations

**Key DocType:**
- **Crispy Print:** Stores print format data (doctype, template name, module, language)
  - `typst_preamble` - Editable Typst header code
  - `typst_layout` - Generated Typst markup (hidden)
  - `layout_json` - Internal JSON representation (hidden, similar to Print Format Builder Beta)

**Key Pages:**
- `/app/crispy-print-builder` - 4-column print format builder
- `/app/crispy-print` - Landing page for templates and formats

## Critical Integration Patterns

### 1. Dual-Mount Vue Architecture

**Two modes for development:**

```bash
# Vite dev mode (hot reload, faster iteration)
cd apps/crispy_print/frontend
yarn dev  # Accessible at http://fdev.local:8080/crispy

# Frappe integration (full context, Socket.io, auth)
bench start  # Test full integration
```

**Build for production:**
```bash
bench build --app crispy_print  # Outputs to crispy_print/public/frontend/
```

### 2. Four-Column Builder Layout

**Reference:** `apps/crispy/public/js/preview/previewPane.js` for layout patterns.

```vue
<!-- App.vue or Builder.vue -->
<template>
  <div class="crispy-builder-grid">
    <!-- Column 1: DocType Fields Palette -->
    <FieldsPane :doctype="currentDoctype" />
    
    <!-- Column 2: Layout Builder (drag-drop sections/fields) -->
    <BuilderPane v-model:layout="layout" />
    
    <!-- Column 3: Live Typst SVG Preview -->
    <PreviewPane 
      :layout="layout" 
      :settings="pageSettings"
      @pdf-ready="handlePdfReady" />
    
    <!-- Column 4: Page Settings (margins, fonts, etc.) -->
    <SettingsPane v-model:settings="pageSettings" />
  </div>
</template>

<style scoped>
.crispy-builder-grid {
  display: grid;
  grid-template-columns: 200px 1fr 1fr 250px;
  gap: 12px;
  height: calc(100vh - 100px);
}
</style>
```

**Column responsibilities:**
1. **Fields:** Available doctype fields from `frappe.get_meta(doctype).fields`
2. **Builder:** Sections/columns/fields structure (Beta builder JSON format)
3. **Preview:** Embed SVG output from Typst worker
4. **Settings:** Page size, margins, fonts, letterhead selection

### 3. Typst Worker Integration

**Reference:** `apps/crispy/public/js/worker/` directory.

Typst compilation runs in a **Web Worker** to prevent UI blocking:

```javascript
// Example: PreviewPanel.vue composable
import { ref, watch } from 'vue';

const worker = new Worker('/assets/crispy_print/typst-worker.js');
const svgOutput = ref(null);
const isCompiling = ref(false);

watch(() => props.layout, (newLayout) => {
  isCompiling.value = true;
  worker.postMessage({
    type: 'compile',
    layout: newLayout,
    settings: props.settings
  });
});

worker.onmessage = (e) => {
  if (e.data.type === 'svg') {
    svgOutput.value = e.data.svg;
    isCompiling.value = false;
  }
};
```

**Translation Pipeline:**
1. Vue layout data → JSON format (Beta builder compatible)
2. JSON → Typst markup (see `apps/crispy/public/js/translator/betaJSONToTypst.js`)
3. Typst CLI compilation → SVG output
4. Display SVG in preview pane

### 4. Socket.io Integration

Socket connection configuration from `sites/common_site_config.json`:

```javascript
// socket.js
import { socketio_port } from "../../../../sites/common_site_config.json";

const protocol = window.location.protocol === "https:" ? "https" : "http";
const host = window.location.hostname;
const port = socketio_port ? `:${socketio_port}` : "";
const siteName = window.frappe?.boot?.sitename || "default";

const url = `${protocol}://${host}${port}/${siteName}`;
socket = io(url, { withCredentials: true });
```

Make available globally: `app.config.globalProperties.$socket`

### 5. Frappe API Integration Patterns

**Backend whitelisted API:**
```python
# crispy_print/api.py
import frappe

@frappe.whitelist()
def generate_typst_pdf(doc_type, doc_name, template_name):
    """Generate PDF using Typst CLI."""
    doc = frappe.get_doc(doc_type, doc_name)
    template = frappe.get_doc("Crispy Print", template_name)
    
    # Your Typst compilation logic
    return {"pdf_url": "/files/output.pdf", "status": "success"}
```

**Frontend resource pattern (preferred):**
```vue
<script setup>
import { createResource } from "frappe-ui";

const generatePDF = createResource({

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agatho-daemon/crispy_print](https://github.com/agatho-daemon/crispy_print) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
