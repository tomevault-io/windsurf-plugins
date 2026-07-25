---
trigger: always_on
description: This notebook guides AI assistants through a step-by-step chain-of-thought to generate a full SQL Database Application using the SQL project SDK, Microsoft.Build.Sql
---


# Guidelines for AI Assistants: Scaffolding SQL Database Projects for T/SQL Applications

This notebook guides AI assistants through a step-by-step chain-of-thought to generate a full SQL Database Application using the SQL project SDK, Microsoft.Build.Sql

Each database object **must** be placed in its own `.sql` file, organized into a coherent folder structure. (this makes it easier for VSCode agent-mode to work with files)

The application should expose Stored Procedures and Views for all operations (all tables should be hidden as internal to the application).

NOTES:
- You (the AI Agent) **must not** edit this file.

## Summary

- Use SQL Database Projects (.sqlproj) to manage schema as code. (Target SQL Server 2019, as this is the latest version that tSQLt supports via a .dacpac)
 - You can find the full documentation for `Sql Database Projects` here: 
  - https://raw.githubusercontent.com/MicrosoftDocs/sql-docs/refs/heads/live/docs/tools/sql-database-projects/sql-database-projects.md
 - Keep T/SQL scripts idempotent and version-controlled.
- For local deployment, use `sqlcmd create mssql` to create a local SQL Server instance. 
 - use the helper script `create-mssql-instance-using-sqlcmd.ps1` in the `./.github/tsql/install` folder
  - To get the connection string, run `sqlcmd config connection-strings` (use the helper script `get-sql-connection-string.ps1` in the `./.github/tsql/install` folder).

## Considerations

- Folder structure maps directly to SQL object types.
- Idempotency: scripts must handle repeated executions.
- Naming conventions align with best practices.
- After each step, you **MUST**:
  - build (using `Build.ps1`, which takes `-ProjectName` as the single parameter)
    - `Build.ps1` is in the `./.github/tsql/inner-loop` folder.
  - publish (using `Publish.ps1`, which takes `-ProjectName` as the single parameter)
    - `Publish.ps1` is in the `./.github/tsql/inner-loop` folder. 
    - `Publish.ps1` uses `SqlPackage` to publish the project to the SQL Server.

NOTES:
- **NEVER** output VSCode.Cell elements in the chat window.  This will cause the chat to break, instead **Always** put output into files in the workspace (as instructed).
- **DO NOT** ask me if I want to do something OR something else.  Until you've completed the last step in this chain of thought, you must not ask me if I want to do something else.  Just do each step as instructed.

## Project Structure

---
```text
MyDatabaseProject/               -- root of .sqlproj
  MyDatabaseProject.sqlproj      -- SQL Database Project file
  Properties/                    -- Database settings and publish profiles
  Tables/                        -- CREATE TABLE scripts
  Views/                         -- CREATE VIEW scripts
  Programmability/               -- Functions, Stored Procedures
    Functions/
    Stored Procedures/
  PreDeploy/                     -- Pre-deployment scripts
  PostDeploy/                    -- Post-deployment scripts and data migrations
  Tests/                         -- tSQLt or other test scripts
```
---

NOTES: 
1. **DO NOT*** put any spaces in the folder names, or in the file names.  Use PascalCase for folder names and file names.
2. **DO NOT** create `.keep` files, they are not needed.
3. You **MUST NOT** put square brackets, `[` and `]`, in file names or folder names.
4. You **MUST NOT** ever add GO statements to any SQL scripts you generate.
5. All Primary Key ID columns must use IDENTITY(1,1)
6. Table IDs must never be used as Stored Procedures parameters
7. **NEVER** use table valued parameters (TVPs) as Stored Procedure parameters (that will be used as MCP Tools), this is because Data Api Builder does not support table valued parameters in entities.

### Step 0: Thoroughly review the business-process-description.md file

**Thought:** Understand the business process and the entities involved, and look for inconsistencies between the Roles, Entities, Entity Descriptions, Operations and User Stories.
- Look for any missing entities, entity descriptions, or missing operations in the business-process-description.md file.
- Look for any missing User Stories in the business-process-description.md file.

**Action:**
- In the agent-mode chat window, strongly suggest any missing entities, entity descriptions, or operations or user stories to the user.  Give very clear reasons why you feel something is missing, especially if you think there is an inconsistency between the Roles, Entities, Entity Descriptions, Operations and User Stories.
- Do not edit the business-process-description.md file directly, just suggest changes in the chat window.
- Particularly look for missing user stories, especially negative cases, i.e. user stories that test unintended behavior by the roles.

### Step 1: Ensure all dependencies are installed

**Thought:** Ensure all dependencies are installed for the SQL Database Project.  Make sure you run these actions one at a time (don't try to concatenate them with ; or &).

**Action:**
- Ensure a container runtime (e.g. docker or podman) is running by running `./.github/tsql/install/is-container-runtime-running.ps1`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/SQL-AI-samples](https://github.com/Azure-Samples/SQL-AI-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
