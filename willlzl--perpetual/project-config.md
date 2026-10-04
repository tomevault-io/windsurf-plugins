---
trigger: always_on
description: Read `docs/pipeline-ui.md` before changing the Pipeline canvas, stage cards, Build and Production, the branch selector or the Git graph.
---

# UI preferences

Read `docs/pipeline-ui.md` before changing the Pipeline canvas, stage cards, Build and Production, the branch selector or the Git graph.

- Keep interface copy minimal: titles, field labels, actions, concise state and actionable errors. Add explanatory paragraphs, helper copy, subtitles or redundant labels only when the user explicitly asks for them.
- Use actual shadcn components for interface controls, and keep the neutral black and gray theme.
- Keep authentic provider brand colors; in the dark theme, invert only monochrome marks.
- Show a stage's configuration or execution status in a native shadcn Badge to the right of the stage title, never in a separate status subtitle row.
- The pipeline inspector has no Details tab.
- Enter Pipeline with the left navigation collapsed; users expand it manually.
- Keep one Pipeline graph layout for every entry URL. Place a prominent native shadcn branch Select at the upper left of the canvas, listing the repository's actual branches through the GitHub connection. Never infer Production from a branch name.
- Let stage cards grow with visible content: wrap long labels and use natural height without internal scrolling. Place following stages from measured widths, preserve zoom during nested expansion, and keep manual Fit View. The canvas toolbar holds zoom and Fit View only, with no Jump to stage control.
- Present stage actions and nested workflow steps on a continuous vertical rail, with circular marks on the left and content on the right, composed from the official Item, Separator and Collapsible components.
- Build and Production start expanded, showing their provider rows; every provider group and its nested content starts collapsed.
- Build holds only the workflow runner, labelled GitHub Actions. Production holds the actual deployment targets the repository configures, such as Railway and Vercel: they are its production deployments, never Build steps. A discovered application directory, such as `frontend`, stays in the underlying scan and is never promoted to a deployment step.
- Render backend delivery groups as supplied: infer no provider groups and hide no deployment targets. Keep workflow runners distinct from deployment targets, and preserve explicit configuration and binding evidence. The deployments GitHub records for the scanned commit join their provider's group in Production as the reporting app's account; a record for another commit never counts, and none changes the stage Badge.
- A stage with an Autopilot record carries it: a Badge beside the status Badge shows its mode, `Autopilot` (merge once verified, the default) or `Ask first`, or the work under way, and opens a native shadcn Dropdown Menu for the mode. A change under way lights the Magic UI Border Beam around the card, lists its steps on the rail, expanded, and offers Stop; a failed workflow row offers Repair for the watched head's failed run. The interface shows only the modes and changes the controller records and fabricates no progress; today only Build records any, for a managed GitHub source.
- Group GitHub workflow actions under one provider card, with a nested native shadcn Collapsible list of workflow, job and step names. Offer no workflow selection or detailed YAML settings.
- Keep discovered Vercel project previews in a separate expandable Vercel provider group in Production, while every workflow stays under GitHub Actions. Discovery does not imply authorized cloud access.
- The Railway drawer holds only configuration file links, with no read-only Build and Deploy field sections.
- Offer Add test only in Sandbox stages, such as Beta and Gamma.
- Beta and Gamma cards show complete business journeys, each with its own queued, running or result state. Clicking a journey focuses its expanded live card; Review and Edit stay explicit actions. Environment readiness never implies test success.
- The environment inspector has exactly two tabs, Integration tests and Runs, and no environment settings page or header gear. Build it from shadcn Sheet, Tabs, Item, Collapsible, Button and Badge primitives, without explanatory subtitles.
- Sandbox stage settings only rename the stage. The stage card footer holds the single delete entry: deletion is confirmed, and the stage's owned sandboxes are cleaned up before the stage is removed.
- Model and API key configuration lives on the app-wide Settings page, reached from the main sidebar and independent of repository or stage selection. Target URL editing stays beside the application link, and the optional test focus stays with Generate.
- App Settings is OpenRouter-only: label the credential OpenRouter API Key, link to API key creation, and offer a native shadcn model Select backed by the actual eligible OpenRouter catalog, with a default preselected, and a second Select for the Escalation model build repairs escalate to, with a strong model the catalog lists preselected. Expose no model ID text input, provider endpoint or Advanced section, and keep a simple layout without nested cards.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [willlzl/Perpetual](https://github.com/willlzl/Perpetual) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
