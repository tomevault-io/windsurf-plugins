---
trigger: always_on
description: When reviewing a **Pull Request (PR)** in this repository, first **determine the type of change** being proposed.
---

# Copilot Instructions: PR Review in This Repository

When reviewing a **Pull Request (PR)** in this repository, first **determine the type of change** being proposed. 

There are two distinct review strategies:

---

## 🟩 **1. If the PR Adds or Modifies a Tool/Service JSON in the `data/` Directory**

You are tasked with reviewing the **new tool or service proposed in the current Pull Request**. The tool is described in a JSON file that adheres to our catalog's schema for **research software quality tools and services**.

Your goal is to:
1. **Verify the tool's eligibility for inclusion** using the criteria below.
2. **Suggest improvements** to the metadata, especially the description, tags, and newly required fields like `quality_dimensions`.
3. **Explain your suggestions and insights** so that:
   - The **PR author understands how to improve the submission**.
   - The **human reviewer can quickly assess** whether to accept or request changes.

You may use the **#web search** feature to gather additional information on the tool.

> ✅ Example of the expected JSON schema:  
> ```json
> {
>   "@context": "https://w3id.org/everse/rs#",
>   "@id": "https://example.org/howfairis",
>   "@type": "SoftwareApplication",
>   "name": "howfairis",
>   "description": "Command line tool to analyze a GitHub or GitLab repository's compliance with the fair-software.eu recommendations",
>   "url": "https://github.com/fair-software/howfairis",
>   "identifier": "https://example.org/howfairis",
>   "isAccessibleForFree": true,
>   "hasQualityDimension": { "@id": "dim:fairness", "@type": "@id" },
>   "howToUse": [ "CI/CD", "command-line" ],
>   "license": "https://spdx.org/licenses/Apache-2.0",
>   "applicationCategory": { "@id":"rs:PrototypeTool", "@type": "@id" }
> }
> ```

### ✅ Step 1: Check Inclusion Criteria

All tools/services added to the TechRadar must satisfy these criteria:

1. Designed to measure and/or improve software quality (and demonstrably does so)
2. Frequently used on research software and actively maintained
3. Enables adherence to relevant research community standards and best practices
4. Has capabilities to analyse and improve software quality across the [research software lifecycle](https://everse.software/RSQKit/life_cycle#the-research-software-lifecycle) (from development to long-term sustainability)


### ✅ Step 2: Review and Suggest Metadata Improvements

#### Metadata to Review and Improve:
- **`name`**: Ensure the tool name is accurate and properly formatted.
- **`description`**: Improve it to:
  - Clearly state how the tool contributes to (research) software quality.
  - Indicate which quality aspects it targets.
- **`url`**: Validate the links.
- **`hasQualityDimension`**: check if the tool's capabilities match the quality dimensions it claims to address. If not, propose a list based on the tool’s capabilities from the following list:

| Quality Dimension      | Description | Sub-characteristics | Source |
|------------------------|-------------|---------------------|--------|
| **compatibility** | Degree to which a product, system or component can exchange information with others and perform its required functions while sharing a common environment. | - Co-existence: Performs functions efficiently in shared environments without negative impacts.  - Interoperability: Can exchange and use information with other products. | [ISO/IEC 25010](https://iso25000.com/index.php/en/iso-25000-standards/iso-25010) |
| **fairness** | Degree to which research software adheres to FAIR principles: Findable, Accessible, Interoperable, Reusable. | None | [FAIR Principles for Research Software](https://www.nature.com/articles/s41597-022-01710-x) |
| **flexibility** | Degree to which a product adapts to changing requirements, contexts, or environments. | - Adaptability  - Scalability  - Installability  - Replaceability | [ISO/IEC 25010](https://iso25000.com/index.php/en/iso-25000-standards/iso-25010) |
| **functional_suitability** | Degree to which a product provides functions that meet stated and implied needs under specified conditions. | - Functional completeness  - Functional correctness  - Functional appropriateness | [ISO/IEC 25010](https://iso25000.com/index.php/en/iso-25000-standards/iso-25010) |
| **interaction_capability** | Degree to which a product can be interacted with by users via the user interface to complete tasks in various contexts. | - Appropriateness recognizability  - Learnability  - Operability  - User error protection  - User engagement  - Inclusivity  - User assistance  - Self-descriptiveness | [ISO/IEC 25010](https://iso25000.com/index.php/en/iso-25000-standards/iso-25010) |
| **maintainability** | Degree of effectiveness and efficiency with which a product can be modified to improve it, correct it, or adapt to changes. | - Modularity  - Reusability  - Analysability  - Modifiability  - Testability | [ISO/IEC 25010](https://iso25000.com/index.php/en/iso-25000-standards/iso-25010) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EVERSE-ResearchSoftware/TechRadar](https://github.com/EVERSE-ResearchSoftware/TechRadar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
