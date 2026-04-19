---
trigger: always_on
description: - **Protocol Key**: `meta_surgeon_protocol`
---

# Meta Surgeon Protocol - Sprint Plan Action Card Definition

## Protocol Identity
- **Protocol Key**: `meta_surgeon_protocol`
- **Mission Title**: Meta Surgeon Protocol
- **Display Name**: Meta Surgeon Protocol
- **Protocol Type**: schema-based
- **Entity Label**: Entity Signal Strength
- **Total Steps**: 4
- **Sprint Circle Index**: 0

## Description
Establish entity signal through structured data implementation. Force search engines to recognize the business as a verified local entity by injecting 4 "Truth Layers" (Organization schema, GeoCircle definitions, Service/Product catalog, and Review aggregation).

## Page 1: Strategist Insight

### Insight Message
"Before we fight for keywords, we must introduce you to the algorithm. Right now, Google is guessing who you are. By injecting these 4 'Truth Layers' into your code, we force the search engines to recognize {{COMPANY_NAME}} as a verified local entity, not just another website."

### Dynamic Placeholders
- `{{COMPANY_NAME}}`: Replaced with `window.currentPropertyName` or property domain name
- `{{PROPERTY_NAME}}`: Same as COMPANY_NAME for this protocol

## Step 1: Global Identity

### Step Definition
```yaml
title: "Global Identity"
description: "First, we hard-code your brand's DNA (Logo, Phone, Social Profiles) into the pages of your site."
```

### Execution Instructions
```yaml
concept: "global brand identity elements"
action: "Add organization schema markup with logo, contact information, and social media profiles"
schemaType: "Organization schema (schema.org/Organization)"
implementation: "Inject structured data into all pages to establish brand identity in search engines"
deliverable: "global-identity-plan.md"
```

### Cursor Prompt Generation Template
```
You are implementing Global Identity as part of Meta Surgeon Protocol.

OBJECTIVE: Add organization schema markup with logo, contact information, and social media profiles

IMPLEMENTATION FOCUS:
Inject structured data into all pages to establish brand identity in search engines

SCHEMA TYPE: Organization schema (schema.org/Organization)

INSTRUCTIONS:
1. Analyze the current website structure in this workspace
   - Detect the technology stack (HTML, React, Vue, Next.js, static site, etc.)
   - Identify all pages/components where global brand identity elements should be implemented
   - Determine the best location for schema markup injection

2. Create a comprehensive implementation plan that includes:
   - Current state analysis: What structure currently exists?
   - Files that need modification: List specific files and their paths
   - Code additions required: Outline the schema markup structure
   - Implementation approach: How to integrate with existing code
   - Dependencies and order: What needs to be done first?
   - Testing strategy: How to verify the implementation works

3. Adapt to the detected technology:
   - For static HTML: Add JSON-LD script tags to <head> or before </body>
   - For React/Vue/Next: Create reusable schema components or use head management
   - For template engines: Inject schema through layout templates
   - For CMSs: Provide plugin recommendations or custom code injection

CONTEXT & REQUIREMENTS:
- Work with ANY file structure and technology stack
- Use schema.org vocabulary for maximum compatibility
- Ensure JSON-LD format for easy implementation
- Make schema dynamic (pull from site data, not hardcoded)
- Follow Google's Structured Data guidelines
- Include: Organization name, logo, contact info (phone, email), social media profiles
- Ensure mobile responsiveness and accessibility
- Validate schema markup can be tested with Google's Rich Results Test

DELIVERABLE:
Create a detailed implementation plan saved as: global-identity-plan.md

The plan should be actionable, technology-agnostic, and ready for immediate implementation regardless of the website's industry (e-commerce, local service, SaaS, restaurant, etc.).
```

## Step 2: Territory Claim

### Step Definition
```yaml
title: "Territory Claim"
description: "Next, we draw the digital borders. We will define the exact 'GEO Circle' for each city you service so Google knows exactly where your trucks go."
```

### Execution Instructions
```yaml
concept: "geographic service area definitions"
action: "Define service areas using geographic schema markup"
schemaType: "GeoCircle or ServiceArea schema (schema.org/GeoCircle)"
implementation: "Specify exact locations where products/services are offered using latitude, longitude, and radius"
deliverable: "territory-claim-plan.md"
```

### Key Implementation Details
- Use GeoCircle schema to define service radius
- Include latitude/longitude coordinates for service center
- Specify radius in kilometers or miles
- Can have multiple service areas for different locations
- Nest within Organization or LocalBusiness schema

## Step 3: Commercial Definition

### Step Definition
```yaml
title: "Commercial Definition"
description: "Now, we define the product. We explicitly tell the bot that 'Roofing' is a Service you sell, with a specific price range, turning your page from a 'brochure' to a 'catalog'."
```

### Execution Instructions
```yaml
concept: "structured product/service catalog with pricing"
action: "Transform content pages into catalog entries with Service/Product schema"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/k4d1e) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
