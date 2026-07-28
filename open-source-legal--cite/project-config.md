---
trigger: always_on
description: When working on pdf annotations or pdf rendering, consult this high level overview
---

# Understanding the PDF Annotation System Architecture

## Admin Tasks if you Edit PDF-related frontend code
1. When working on document knowledge base and pdf-related frontend functionality, ensure you update our cursor rule here [pdf-viewer-and-annotator-architecture.mdc](mdc:.cursor/rules/pdf-viewer-and-annotator-architecture.mdc) such that it captures any material changes to our existing setup.
2. When modifying filtering logic, ensure changes are applied consistently to both `useVisibleAnnotations` and `useVisibleRelationships` hooks to maintain unified behavior.

## Context: Document Knowledge Base

The `DocumentKnowledgeBase` component serves as the main container for viewing and interacting with documents, particularly PDFs. It provides a comprehensive interface that combines document viewing with annotation capabilities, search functionality, knowledge extraction features, and a sophisticated dual-layer architecture for both document annotation and knowledge synthesis.

The system uses a sophisticated virtualized rendering approach for PDFs, where only visible pages (plus a small overscan buffer) are rendered to dramatically improve performance with large documents. This is coupled with an advanced annotation management system that allows users to view, filter, and interact with annotations overlaid on the document.

## Intent: High-Performance Document System with Dual-Layer Architecture

The architecture aims to:

1. Efficiently render large PDF documents using virtualization (only visible pages are rendered)
2. Provide a flexible annotation system that can display/hide annotations based on user preferences
3. Support various annotation types (spans, tokens, structural elements)
4. Enable real-time filtering and selection of annotations
5. Maintain consistent state across the entire application using Jotai atoms
6. Ensure smooth scrolling and navigation even with hundreds of pages
7. Offer dual-layer architecture for document viewing and knowledge synthesis
8. Provide unified content feeds combining notes, annotations, and relationships
9. Support version-controlled document summaries with version history
10. Enable flexible, resizable panel layouts with auto-minimize capabilities

## Dual-Layer Architecture

The system implements two distinct viewing layers:

### Document Layer
- Traditional PDF/text viewing with annotations
- Search functionality with highlighting
- Annotation creation and editing
- Extract and analysis results
- Real-time collaboration features

### Knowledge Layer
- Document summary viewing and editing
- Version history with author tracking
- Markdown-based content editing
- Clean reading experience
- Knowledge synthesis tools

Users can seamlessly switch between layers based on their current task, with intelligent context-aware switching.

## Component Interactions

### State Management with Jotai

The system uses Jotai atoms to manage global state:

```
AnnotationAtoms.tsx
├── pdfAnnotationsAtom - Core state (annotations, relations, docTypes)
├── structuralAnnotationsAtom - Structural annotations
├── allAnnotationsAtom - Computed, de-duplicated annotation list
└── perPageAnnotationsAtom - Page-indexed annotation map

UISettingsAtom.tsx
├── activeLayerAtom - Current layer (knowledge/document)
├── chatPanelWidthAtom - Panel width management
├── sidebarViewModeAtom - Chat vs feed mode
├── showStructuralAtom - Show/hide structural content
├── showSelectedOnlyAtom - Filter to selected items only
├── showBoundingBoxesAtom - Visual display setting
└── showLabelsAtom - Label display behavior

DocumentAtom.tsx
├── scrollContainerRefAtom - Scroll container reference
├── pendingScrollAnnotationIdAtom - Annotation to scroll to
└── textSearchStateAtom - Search state management
```

This approach provides:
- Centralized state management
- Computed derivations that automatically update
- Efficient re-rendering when only relevant state changes
- Layer-aware state transitions

### PDF Rendering Pipeline

1. **DocumentKnowledgeBase** initializes the document view and loads annotation data via GraphQL
   - Loads PDF document using pdfjs-dist
   - Fetches PAWLS token data for precise text selection
   - Initializes Jotai atoms with annotation data
   - Manages layer switching and tab navigation

2. **PDF** component manages the virtualized rendering of pages:
   - Calculates page heights at current zoom level (cached per zoom)
   - Uses binary search to determine visible page range
   - Adds overscan buffer (2 pages above/below) for smooth scrolling
   - Ensures selected annotation/search result pages are always rendered
   - Absolutely positions all pages based on cumulative heights

3. **PDFPage** renders individual pages with their annotations:
   - Only mounts when page is in visible range
   - Creates and manages its own PDF canvas
   - Overlays annotation components on top of the canvas
   - Handles user interactions for creating/selecting annotations
   - Implements two-phase scroll-to-annotation system

### Advanced Features

#### Unified Content Feed
The `UnifiedContentFeed` component provides a consolidated view of all document content:
- Combines notes, annotations, relationships, and search results
- Sortable by page order or chronologically

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-Source-Legal/cite](https://github.com/Open-Source-Legal/cite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
