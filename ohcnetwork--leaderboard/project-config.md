---
trigger: always_on
description: This document provides specialized workflows for different types of tasks in the leaderboard system. Each agent represents a specific role with its own workflow, considerations, and best practices.
---

# Leaderboard Project - Specialized Agent Workflows

This document provides specialized workflows for different types of tasks in the leaderboard system. Each agent represents a specific role with its own workflow, considerations, and best practices.

---

## Agent: Plugin Developer

**Purpose**: Create and test new data source plugins for the leaderboard system.

### Workflow

1. **Scaffold Plugin Project**

   ```bash
   pnpm create-leaderboard-plugin <path>
   # Example: pnpm create-leaderboard-plugin ../leaderboard-github-plugin
   ```

   The CLI will prompt for:
   - Plugin name (e.g., 'github', 'slack', 'jira')
   - Plugin description
   - Author name

   This generates a complete project structure with:
   - `package.json` with correct dependencies
   - `tsconfig.json` with proper configuration
   - `vitest.config.ts` for testing
   - `src/index.ts` with plugin template
   - `src/__tests__/plugin.test.ts` with test examples
   - `README.md` with documentation

2. **Implement Setup Method**

   Define activity types in the `setup()` method:

   ```typescript
   import { activityDefinitionQueries } from "@ohcnetwork/leaderboard-api";

   async setup(ctx: PluginContext): Promise<void> {
     // Define all activity types your plugin will track
     await activityDefinitionQueries.upsert(ctx.db, {
       slug: "pr_opened",
       name: "Pull Request Opened",
       description: "Opened a pull request",
       points: 5,
       meta: { icon: "git-pull-request" },
     });

     await activityDefinitionQueries.upsert(ctx.db, {
       slug: "pr_merged",
       name: "Pull Request Merged",
       description: "Had a pull request merged",
       points: 10,
       meta: { icon: "git-merge" },
     });
   }
   ```

3. **Implement Scrape Method**

   Fetch and store activities:

   ```typescript
   import { activityQueries, contributorQueries } from "@ohcnetwork/leaderboard-api";

   async scrape(ctx: PluginContext): Promise<void> {
     const { apiToken, organization } = ctx.config;

     // Fetch data from external API
     const prs = await fetchPullRequests(apiToken, organization);

     for (const pr of prs) {
       // Ensure contributor exists
       await contributorQueries.upsert(ctx.db, {
         username: pr.author.login,
         name: pr.author.name,
         role: "contributor", // or determine from your logic
         avatar_url: pr.author.avatar_url,
       });

       // Store activity
       await activityQueries.create(ctx.db, {
         slug: `pr-${pr.id}`,
         contributor: pr.author.login,
         activity_definition: pr.merged ? "pr_merged" : "pr_opened",
         title: pr.title,
         occurred_at: pr.created_at,
         link: pr.html_url,
         points: pr.merged ? 10 : 5,
         meta: {
           repository: pr.repository,
           additions: pr.additions,
           deletions: pr.deletions,
         },
       });
     }

     ctx.logger.info(`Processed ${prs.length} pull requests`);
   }
   ```

4. **Implement Aggregate Method (Optional)**

   Compute plugin-specific aggregates after the main leaderboard aggregation:

   ```typescript
   import { activityQueries, contributorQueries, contributorAggregateQueries } from "@ohcnetwork/leaderboard-api";

   async aggregate(ctx: PluginContext): Promise<void> {
     const contributors = await contributorQueries.getAll(ctx.db);

     for (const contributor of contributors) {
       const activities = await activityQueries.getByContributor(ctx.db, contributor.username);
       const mergedPRs = activities.filter(a => a.activity_definition === "pr_merged");

       await contributorAggregateQueries.upsert(ctx.db, {
         aggregate: "pr_merged_count",
         contributor: contributor.username,
         value: { type: "number", value: mergedPRs.length, format: "integer" },
         meta: { calculated_at: new Date().toISOString() },
       });
     }

     ctx.logger.info("PR merge count aggregates computed");
   }
   ```

   > The `aggregate()` method runs after the main leaderboard aggregation, so standard aggregates like `total_activity_points` and `activity_count` are already available.

5. **Use Query Builders**

   Always use provided query builders from the API package:
   - `contributorQueries`: create, upsert, getByUsername, getAll, etc.
   - `activityQueries`: create, getByContributor, getByDateRange, etc.
   - `activityDefinitionQueries`: upsert, getBySlug, getAll, etc.

6. **Write Tests**

   Create comprehensive tests in `src/__tests__/plugin.test.ts`:

   ```typescript
   import { describe, it, expect, beforeEach, afterEach } from "vitest";
   import {
     createDatabase,
     initializeSchema,
   } from "@ohcnetwork/leaderboard-api";
   import plugin from "../index";

   describe("My Plugin", () => {
     let db: Database;

     beforeEach(async () => {
       db = createDatabase(":memory:");
       await initializeSchema(db);
     });

     afterEach(async () => {
       await db.close();
     });

     it("should define activity types in setup", async () => {
       const ctx = {
         db,
         config: {},
         orgConfig: {
           /* mock org config */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohcnetwork/leaderboard](https://github.com/ohcnetwork/leaderboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
