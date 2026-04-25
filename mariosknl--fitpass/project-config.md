---
trigger: always_on
description: Anything related to Sanity usage with Typescript & App SDK
---


# App SDK and TypeGen

> [!NOTE]
> This page is for using TypeGen with the App SDK. See [the TypeGen article](/docs/apis-and-sdks/sanity-typegen) to generate types for your Studio and front end applications.

[Sanity TypeGen](/docs/apis-and-sdks/sanity-typegen) is a tool that generates TypeScript types directly from your Sanity schemas and GROQ queries. When used with the Sanity App SDK, it provides strong type safety and autocompletion suggestions for your documents, query results, and projections.

In this guide, we’ll walk through setting up and using TypeGen within your SDK app.

> [!WARNING]
> Experimental feature
> TypeGen support in the App SDK is currently in its early stages. We’re actively working on improving this integration and the developer experience around it. For now, some parts of this process may be suboptimal, but we invite the adventurous among you to follow along!

## Setup

Using Typegen involves two main steps: extracting your schema(s) and then generating the types. Both commands are available via the CLI.

### Extract schemas

First, you need to extract your Sanity schema(s) into a JSON format that Typegen can understand. **Currently, this step relies on the full** **sanity** **package**, typically used within your Sanity Studio project, as Typegen needs access to the complete schema definition to generate accurate types.

Schema extraction is performed within your Studio setup to generate the `schema.json` file. Once created, this file can be used independently by other tools or parts of your workflow.

> [!NOTE]
> We recognize that requiring the Studio environment solely for this generation step isn't ideal, and we're actively working on improving this workflow in future App SDK updates to make the process more self-contained.

Use the `sanity schema extract` command within your Studio project or a project that has the `sanity` package installed:

**Terminal**

```sh
npx sanity schema extract --workspace <workspace-name> --output-path <path/to/schema.json>
```

This `schema.json` file can be copied to (or the `--output-path` can be set directly to) your Sanity app's repository. Your application itself does _not_ need the full `sanity` package as a dependency to use the generated types; it only needs the `schema.json` file for the `typegen generate` step.

If your Studio project defines multiple workspaces or you need types for different schemas (e.g., for different datasets), run the `extract` command for each one, outputting to separate JSON files. For example, you could configure you Studio’s `package.json` as follows:

**package.json**

```json
{
  "scripts": {
    "schema:extract:test": "sanity schema extract --workspace test --output-path ../my-frontend-app/schema-test.json",
    "schema:extract:prod": "sanity schema extract --workspace production --output-path ../my-frontend-app/schema-prod.json",
    "schema:extract": "npm run schema:extract:test && npm run schema:extract:prod"
  }
}
```

We plan to improve this schema extraction process as the SDK matures to potentially reduce the dependencies and improve overall developer experience.

### Install (experimental) packages

To use the Typegen features described in this guide, your SDK app needs specific experimental versions of `@sanity/cli` and `groq` installed. Install these packages from within your SDK app directory:

**Terminal**

```sh
npm install groq@typegen-experimental-2025-04-23
npm install @sanity/cli@typegen-experimental-2025-04-23 --save-dev
```

> [!WARNING]
> Package names and installation
> These are experimental pre-release versions. The package names and installation process may change as these features stabilize.

### Configure TypeGen (optional)

For the most common use case – a single Sanity schema for your project – **no configuration file is needed**. However, you'll need to create a TypeGen configuration file for more complex use cases, such as:

- Using multiple schemas (e.g., from different workspaces or for different datasets).
- Needing to explicitly map a single schema to a specific `schemaId` for accurate schema scoping (instead of using the default `'default'`).
- Using a different name or location for your schema file(s).
- Specifying a custom output path for the generated types file.

If you need this level of configuration, create a TypeGen configuration file (`sanity-typegen.json` ) at the root of your SDK app and use the `unstable_schemas` array:

**sanity-typegen.json**

```json
// sanity-typegen.json
{
  "unstable_schemas": [
    {
      // Path to the schema
      "schemaPath": "./schemas/products-schema.json",
      // The schema ID, formatted as `projectId.datasetName`
      "schemaId": "your-project-id.products"
    },
    {
      "schemaPath": "./schemas/authors-schema.json",
      "schemaId": "your-project-id.authors"
    }
    // Add more schema objects if needed
  ],
  "overloadClientMethods": false // client methods are not needed for the App SDK
  // Optional: Specify output path for generated types
  // "outputPath": "./src/generated/sanity-types.ts"
}
```

Objects in the `unstable_schemas` array each consist of the following properties:

- **schemaPath:** The path (relative to the project root) to the corresponding extracted schema JSON file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mariosknl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
