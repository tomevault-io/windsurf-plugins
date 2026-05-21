---
trigger: always_on
description: Comment styles
---


- Use english comments only
- No comments in code whatsoever unless the code is extremely unusual and impossible to understand without explanation
- Only add comment to a property if the name is not self-explaining.
- Comment the component class using the format explain below.

Each component’s JSDoc should start with a clear, one-line summary that identifies what the component is and its primary purpose. Use the component’s tag name and a brief description (including a common synonym if the name is non-standard). For example:
/\*\*

- `<obc-floating-item>` – A transient toast notification component for brief messages.
-
- ...
  \*/
  This opening line helps developers quickly recognize the component (e.g., “floating message” is essentially a toast/snackbar notification). It also ensures the description contains searchable keywords (like toast, notification, snackbar) so that a RAG system can retrieve it for relevant queries. After the one-liner, include 1-2 sentences expanding on the component’s purpose and context. This should cover the what and why: what the component does and in what scenario it’s used. Emphasize practical use cases without repeating the OpenBridge domain context (assume it is already known).
  **Tone rule:** Do NOT mention “maritime”, “industrial”, “bridge”, or similar environmental qualifiers; keep text domain-agnostic. For example: “Appears temporarily to display non-critical feedback or status updates, floating above the UI so it doesn’t interrupt the user’s workflow.”
  Key Features and Variants
  Use a “Features” section with bullet points (or sub-sections) to highlight the component’s main capabilities, configuration options, and variants. This makes it easy to scan. For instance:
  Variants or Types: List distinct visual/behavioral variants (e.g., regular vs. application messages, checked vs. unchecked states, etc.). If the component has named types (perhaps via an enum), mention each and what it means.
  State or Style Options: Note major style configurations (like horizontal vs. vertical layout, single-line vs. multi-line content, palette variations, size options, etc.).
  Interactive Elements: Include if it supports actions (like buttons or icons) or dynamic content.
  Notable Behaviors: For example, auto-dismiss timing, focus handling, or responsiveness. Anything the component does automatically (or expects from the developer) should be called out here.
  Each bullet should be concise but descriptive. E.g.: “Layout directions: Supports horizontal (side-by-side layout) or vertical (stacked layout) to adapt to available space.” This section gives a quick feature summary at a glance. If the component has multiple distinct modes or sub-variants, you can break them out with subheadings or bold labels for clarity. For example, for a toggle component like obc-check-button, you might have:
  Regular mode: Description…
  Checkbox mode: Description…
  Detail what each mode is meant for and how they differ (as in the user’s draft, where Regular Type vs Checkbox Type are explained in separate paragraphs). This helps a reader understand the nuances of each variant.
  Usage Guidelines and Use Cases

* important! When describing a property, base the explanation strictly on its code usage or story; if the purpose is unclear, insert **TODO(designer)** instead of guessing. This is IMPORTANT for all the documentation. If a bit unsure, write TODO so it's easy to catch and have a designer write the indended purpose.
  After features, provide guidance on when and how to use the component. This can be a short paragraph or a list of use cases. Frame it as advice: what scenarios is this component ideal for, and how it fits into the UI/UX. For example:
  “Use obc-floating-item for brief, transient feedback (e.g., form submissions, status updates). It’s ideal when you need to confirm an action or show a non-critical alert without disrupting the workflow. Avoid using it for persistent or critical alerts – those might require a dialog or an alert banner.”
  If relevant, contrast the component with similar ones to clarify choices. For instance: “Unlike a standard obc-alert banner that stays in the content flow, a floating message is ephemeral and overlays other content.” This helps developers decide if this is the right component for their need, which is crucial for the RAG model to answer “What component should I use for X?”. Include searchable keywords and synonyms in these explanations. (E.g., for a filter chip component, mention terms like “tag”, “pill”, or “token” if those are common synonyms.) This improves discovery via search queries.
  Slots and Content Structure
  For Web Components, document all content slots clearly in a Slots section (preferably as a table for readability). List each slot name, conditions when it’s used, and its purpose:
  Slot Name Renders When... Purpose
  primary-icon Always (for all messages) Main icon to represent the message’s category.
  secondary-icon type="application" only Additional icon for application-type messages.
  title Always Title or heading of the message.
  description Always Detailed message text.
  time If hasTimestamp is true Timestamp label (e.g., “12:45”).
  day If hasTimestamp && hasDay Day label (e.g., “Mon”).
  action If action property is true Label for the primary action button.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ocean-Industries-Concept-Lab/openbridge-webcomponents](https://github.com/Ocean-Industries-Concept-Lab/openbridge-webcomponents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
