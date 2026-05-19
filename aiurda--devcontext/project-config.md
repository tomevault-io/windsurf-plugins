---
trigger: always_on
description: Require using Context7 or Web Search to review library documentation before implementation
---


# External Library Documentation Requirements

- **ALWAYS Use Context7 Before Using External Libraries**

  - The agent MUST retrieve and review documentation via Context7 before implementing any code that uses an external library
  - This applies to ALL libraries not part of the standard language libraries
  - No exceptions - even for commonly known libraries like React, Express, or Lodash

- **Two-Step Documentation Retrieval Process**

  ```javascript
  // ✅ DO: ALWAYS follow this exact two-step process
  // Step 1: Resolve the library name to a Context7-compatible ID
  const libraryIdResponse =
    await mcp_context7_resolve-library-id({
      libraryName: "express",
    });

  // Step 2: Get the documentation using the resolved ID
  const docsResponse =
    await mcp_context7_get-library-docs({
      context7CompatibleLibraryID: libraryIdResponse.libraryId,
      tokens: 10000, // Adjust based on documentation needs
      topic: "routing", // Optional: focus on specific area
    });

  // ❌ DON'T: Skip the resolution step
  // ❌ DON'T: Use hardcoded library IDs
  // ❌ DON'T: Proceed with implementation without review
  ```

- **Never Skip Documentation Retrieval**

  - Documentation MUST be retrieved even for seemingly simple APIs
  - Do not rely on previously cached knowledge for current implementations
  - Never make assumptions about library interfaces, verify with current documentation

- **Document First, Implement Second**

  ```javascript
  // ✅ DO: Review documentation BEFORE writing implementation code
  // 1. Identify library need
  // 2. Retrieve documentation
  // 3. Review relevant sections
  // 4. THEN implement solution

  // ❌ DON'T: Implementation without documentation
  const app = express(); // WRONG - Documentation not retrieved first
  app.get("/", (req, res) => res.send("Hello"));
  ```

- **Verify API Compatibility**

  - Always check current API version compatibility
  - Validate method signatures against retrieved documentation
  - Verify required dependencies and peer dependencies

- **Handle Documentation Response Properly**

  ```javascript
  // ✅ DO: Properly handle the documentation response
  const docsResponse =
    await mcp_context7_get-library-docs({
      context7CompatibleLibraryID: "vercel/nextjs",
    });

  // Extract relevant information
  const sections = docsResponse.content.sections;
  const examples = sections.filter((s) => s.type === "example");
  const apiDocs = sections.filter((s) => s.type === "api");

  // Use this for implementation guidance
  // ...

  // ❌ DON'T: Ignore retrieved documentation
  // ❌ DON'T: Proceed with implementation based on assumptions
  ```

- **Required Documentation Review Checklist**

  - Core API functions and methods must be verified
  - Method signatures and parameters must be validated
  - Return values and types must be confirmed
  - Required configuration must be identified
  - Common patterns and examples must be analyzed

- **Example Implementation Flow**

  ```javascript
  // ✅ DO: Follow this implementation flow

  // 1. Identify need for external library
  // "I need to implement JWT authentication in Express"

  // 2. Resolve library IDs for ALL needed libraries
  const expressLibrary =
    await mcp_context7_resolve-library-id({
      libraryName: "express",
    });

  const jwtLibrary =
    await mcp_context7_resolve-library-id({
      libraryName: "jsonwebtoken",
    });

  // 3. Retrieve documentation for ALL libraries
  const expressDocs =
    await mcp_context7_get-library-docs({
      context7CompatibleLibraryID: expressLibrary.libraryId,
    });

  const jwtDocs =
    await mcp_context7_get-library -
    docs({
      context7CompatibleLibraryID: jwtLibrary.libraryId,
      topic: "authentication",
    });

  // 4. Review documentation and extract implementation details
  // 5. Create implementation with proper reference to documentation

  // ❌ DON'T: Skip any library in multi-library implementations
  ```

- **Documentation First for Dependency Resolution**

  - All transitive dependencies must be documented
  - Version compatibility must be verified
  - Properly handle conflicting dependencies

- **Update Implementation After Documentation Review**

  ```javascript
  // ✅ DO: Update existing code based on documentation
  // If reviewing code that uses libraries without proper documentation:

  // 1. Retrieve documentation for used libraries
  // 2. Verify existing implementation against documentation
  // 3. Correct any discrepancies found

  // ❌ DON'T: Assume existing implementation is correct
  // ❌ DON'T: Skip verification of existing library usage
  ```

- **MUST Use Web Search When Documentation Is Unavailable**

  - If Context7 cannot provide documentation or returns insufficient information, the agent MUST use the web search tool
  - Always search for the most recent documentation as of mid-2025
  - Verify the library version against the latest available release

  ```javascript
  // ✅ DO: Fallback to web search when Context7 fails
  try {
    // First attempt to use Context7
    const libraryIdResponse =
      (await mcp_context7_resolve-library-id({
        libraryName: "some-library",
      });

    const docsResponse =

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aiurda/devcontext](https://github.com/aiurda/devcontext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
