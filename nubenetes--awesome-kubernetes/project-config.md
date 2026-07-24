---
trigger: always_on
description: This file contains the accumulated instructions and long-term vision for the autonomous maintenance of Nubenetes.com. AI agents must consult this document in every iteration to ensure learning continuity.
---

# Nubenetes Intelligent Curation: Meta-Instructions & Learning Roadmap

This file contains the accumulated instructions and long-term vision for the autonomous maintenance of Nubenetes.com. AI agents must consult this document in every iteration to ensure learning continuity.

## 🧠 Core Mandates

1.  **Information Preservation**: NEVER delete summaries, comments, or stars (🌟) accompanying links. The bot should only update the URL or reorganize the item's position, never delete the descriptive context.
2.  **Persistent Learning**: Use `src/memory/health_learning.json` to store knowledge about domains (anti-bot blocks, successful strategies) and navigation patterns.
3.  **Minimum Viable Quality (MVQ)**: For GitHub/GitLab repositories, the bot MUST check the last commit date. If the repository has had NO activity (commits) in more than **4 years**, it must receive a significantly lower `impact_score` and be deprioritized, even if the content remains technically relevant. This ensures Nubenetes stays fresh and focuses on maintained projects.
4.  **Style Guide (High-Density Summaries)**: All injected summaries MUST follow a **High-Density Descriptive** style. Avoid generic "clickbait". Instead, apply the **Double-Evidence Synthesis Protocol**: contrast 'Curator Insight' with 'Live Grounding' (MCP) to provide a neutral, professional description of architectural value, key features, and technical significance. Summaries should be 2-5 sentences long and support multi-line Markdown formatting (bullet points).
5.  **Semantic Interlinking**: The bot should identify related categories for each resource. While the full entry is injected into the primary category, a short reference (*"See also: [Title](URL) in [Category]"*) should be added to up to two related categories to improve site navigation.
6.  **Visual Health Dashboard**: Every curation run MUST generate a local `report.html` (outside the repo) for visual validation of metrics, quality (MVQ), and AI decisions.
7.  **Total Resilience**: The workflow must be able to continue even if there are individual errors in link or file validations. Prioritize generating a result (PR) even if it is partial.
8.  **Repository Consolidation & Deep-Link Preservation**: In case of a failure (404) in a deep GitHub/GitLab link, the bot SHOULD try to validate the repository root before considering it dead. However, if a deep link (wiki, PR, tree) is ALIVE, it MUST be preserved. We prefer specific technical context in V1.
9.  **URL Expansion**: All shortened links (t.co, bit.ly, buff.ly, etc.) MUST be expanded to their original long version before being evaluated or injected. This ensures inventory homogeneity and improves global deduplication precision.
10. **Linguistic Diversity & Global Access**:
    - **Primary Language**: English is the official language of the Nubenetes ecosystem.
    - **Native Preservation (V1 Archive)**: For non-English resources (e.g., Spanish repositories, videos, or articles), the V1 description MUST remain in the **resource's native language** to preserve its original context and cater to native speakers.
    - **Global Synthesis (V2 Portal)**: To ensure 100% global discoverability, the V2 Elite summaries (`ai_summary`) MUST always be in **Professional English**, regardless of the source language.
    - **V1 Immutability**: For links already present in the V1 archive, AI agents MUST NOT overwrite manually curated titles, stars, or additional descriptive comments. Only broken URLs or missing metadata fields (like year/language) should be updated.
    - **Rich Metadata Enrichment**: AI agents SHOULD attempt to extract technical authors, video durations (YouTube), and reading times (Blogs) to populate high-density dimensions in the V2 portal.
    - **Safety Guard Validation**: Every automated PR MUST undergo syntax validation and a test MkDocs build to prevent broken rendering or security vulnerabilities in the final site.
    - **Explicit Language Tagging**: All non-English resources in the V2 Portal MUST be explicitly tagged (e.g., `[SPANISH CONTENT]`, `[FRENCH CONTENT]`) at the end of the entry to inform global users before they navigate.
    - **English-First Exceptions**: Global software projects (even if created by Spanish speakers) that use English as their primary interface should be curated entirely in English. Native preservation is for localized content like blogs, videos, and guides.
11. **Workflow-Config Synchronization**: The GitHub Actions curation workflow form (`agentic_cron.yml`) MUST remain perfectly synchronized with the curation sources configuration file (`data/curation_sources.yaml`). Any addition, removal, or renaming of topics/categories in the configuration file requires a corresponding update to the workflow's input fields (checkboxes) to ensure users can toggle those sources manually. This maintains consistency between data-driven sources and the UI trigger.
12. **V2 Elite Maintenance**: The Nubenetes V2 (Agentic Elite) edition is a derived view of the V1 archive. It is managed via the `src/v2_optimizer.py` script and stored in the `v2-docs/` directory.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nubenetes/awesome-kubernetes](https://github.com/nubenetes/awesome-kubernetes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
