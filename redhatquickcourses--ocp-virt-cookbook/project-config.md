---
trigger: always_on
description: Project guidelines, AsciiDoc standards, and review rubric for the OpenShift Virtualization Cookbook
---


# OpenShift Virtualization Cookbook - Project Guidelines

This file is the single source of truth for content standards, AsciiDoc conventions, and review criteria. Reviewer personas reference this file directly.

## Branch Management

Git policy (local branches only, no push, no PRs, commit attribution) is enforced by `shared-constraints.mdc`. Additional guidance:

- Create a new branch with a descriptive name for each task
- Keep commits focused on a single logical change
- Use clear commit messages that explain what changed and why

## Communication and Documentation Style

- NEVER use icons or emojis in any documentation, code comments, or communication
- Use clear, professional technical writing; active voice preferred
- Be direct; avoid filler words and overly casual language
- Maintain consistent tone throughout each document
- Follow Antora documentation best practices

## Verification and Research

- Verify information against official OpenShift, OKD, and KubeVirt documentation
- Cross-reference multiple sources when uncertain
- Verify API versions and resource definitions are current for the target OpenShift version (4.18+)

## Documentation Content

### Length and Scope

| Document Type | Target Words | Approx. Read Time |
|--------------|--------------|-------------------|
| Quick Tutorial | 800-1,500 | 5-10 min |
| Standard Tutorial | 1,500-3,000 | 10-20 min |
| Deep Dive | 3,000-5,000 | 20-30 min |
| Reference Guide | 5,000+ | 30+ min |

- Focus on one specific topic or use case per page
- Use xref to related content instead of duplicating
- Avoid scope creep or tangential topics

### Required Tutorial Structure

Every tutorial MUST include these sections in order:

1. **Title and Overview** -- Level-1 heading with `:navtitle:`. Brief overview of what will be accomplished and why.
2. **Prerequisites** -- OpenShift version, CLI tools, prior tutorials, cluster requirements. Version tested inline (e.g., "OpenShift 4.18+ (tested on 4.20)"), not in a separate block.
3. **Main Content** -- Logical flow from simple to complex. One topic per page. `==` and `===` for hierarchy. Verification steps after each major section.
4. **Troubleshooting** (when applicable) -- Common issues and solutions, links to detailed guides.
5. **Cleanup** -- Commands to remove all resources created.
6. **Summary** -- Bullet list of what was learned.
7. **See Also** -- Final section heading must be `== See Also` (not "References"). Internal `xref:`, external `link:` with `window=_blank`.

### Technical Accuracy

- All technical information must be accurate and verifiable
- YAML manifests must be syntactically correct and complete
- Commands must be tested and functional
- API versions must be current for OpenShift 4.18+
- Resource names, namespaces, and fields must be correct
- Cross-reference with official OpenShift and KubeVirt documentation

## AsciiDoc Formatting

### Structural Elements

- Level-1 heading (`=`) at document start
- Proper header hierarchy (no skipping levels)
- Blank lines between sections
- `:navtitle:` attribute set appropriately

### Code Blocks

- Always specify language: `[source,yaml]`, `[source,bash]`, `[source,json]`
- Use `role=execute` for commands users must run: `[source,bash,role=execute]`
- Heredoc blocks (`oc apply -f - <<EOF`) use `[source,bash,role=execute]`, not `[source,yaml]`
- Each code block contains a single command or tightly coupled pipeline
- Host-side commands (`oc`, `virtctl`) and guest-side commands are in separate code blocks with prose explaining the context switch
- Do not use bash comments (`#`) as explanatory prose inside code blocks; use prose between blocks instead
- Output blocks use plain `----` without language tag
- Preserve proper indentation (2-space for YAML)

### Links and References

- Internal: `xref:page-name.adoc[Link Text]` (same module), `xref:module-name:page-name.adoc[Link Text]` (cross-module)
- External: `link:https://...[Text,window=_blank]`
- Downloadable manifests: `xref:attachment$filename.yaml[Download filename.yaml]` immediately above the `[source,yaml]` block

### Admonitions

- Use `NOTE:`, `WARNING:`, `IMPORTANT:`, `TIP:`, `CAUTION:` appropriately
- Do not overuse; reserve for meaningful callouts

### Images

- Stored in module's `images/` directory
- Descriptive alt text: `image::filename.png[Alt Text]`
- Reasonable file size (flag images > 500 KB)

### Tables

- Proper column definitions and header rows
- Consistent formatting; test rendering in preview

## File Organization

### Module Structure

- Place content in appropriate modules under `modules/`
- Each module: `pages/`, `nav.adoc`, optional `images/` and `attachments/`
- Descriptive, kebab-case filenames
- Keep `nav.adoc` updated when adding/removing pages
- Also update the module's `index.adoc` Sections list (if one exists) with a corresponding xref entry

### Manifest Files

- Place YAML manifests in `modules/<module>/attachments/<tutorial-name>/`

## Code and Manifest Quality

- 2-space indentation for YAML
- All required fields: `apiVersion`, `kind`, `metadata`, `spec`
- Descriptive resource names; specify namespaces explicitly
- Comments for non-obvious configuration choices

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedHatQuickCourses/ocp-virt-cookbook](https://github.com/RedHatQuickCourses/ocp-virt-cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
