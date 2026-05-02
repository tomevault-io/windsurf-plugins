---
trigger: always_on
description: <?xml version="1.0" encoding="UTF-8"?>
---

<?xml version="1.0" encoding="UTF-8"?>
<project_knowledge_base>
    <title>PROJECT KNOWLEDGE BASE</title>

    <description>Aether is a VS Code extension that enhances GitHub Copilot Chat with multiple AI providers including ZhipuAI, MiniMax, MoonshotAI, DeepSeek, Codex (OpenAI), Chutes, OpenCode, and custom OpenAI/Anthropic compatible models.</description>

    <section name="STRUCTURE">
        <file_tree>
            <folder name="copilot-helper">
                <folder name="src">
                    <folder name="accounts">Account management</folder>
                    <folder name="copilot">Core Copilot integration</folder>
                    <folder name="providers">AI provider implementations</folder>
                    <folder name="status">Status bar components</folder>
                    <folder name="ui">User interface components</folder>
                    <folder name="utils">Shared utilities</folder>
                    <folder name="utils">
                        <file name="knownProvidersData.ts">Pure provider data — NO vscode imports</file>
                        <file name="knownProviders.ts">Extension logic — imports from knownProvidersData.ts</file>
                    </folder>
                    <folder name="tools">Tool implementations</folder>
                    <folder name="types">Type definitions</folder>
                    <folder name="prompt">AI prompts and instructions</folder>
                </folder>
                <folder name="aether">Aether CLI (src/aether)</folder>
                <folder name="dist">Compiled output</folder>
                <folder name="fonts">Custom fonts</folder>
                <folder name=".vscode">VS Code configuration</folder>
                <file name="esbuild.config.js">Build configuration</file>
                <file name="tsconfig.json">TypeScript configuration</file>
                <file name="biome.config.json">Biome linting/formatting configuration (replaces ESLint)</file>
                <file name="package.json">Extension manifest</file>
                <file name="README.md">Documentation</file>
            </folder>
        </file_tree>
    </section>

    <section name="WHERE_TO_LOOK">
        <table>
            <row>
                <cell>Task</cell>
                <cell>Location</cell>
                <cell>Notes</cell>
            </row>
            <row>
                <cell>Add new AI provider</cell>
                <cell>src/utils/knownProvidersData.ts</cell>
                <cell>Add to knownProviderOverrides, then run npm run sync-providers</cell>
            </row>
            <row>
                <cell>Aether CLI source</cell>
                <cell>src/aether/</cell>
                <cell>Reads from knownProvidersData.ts (no vscode imports)</cell>
            </row>
            <row>
                <cell>Account management</cell>
                <cell>src/accounts/</cell>
                <cell>Multi-account support</cell>
            </row>
            <row>
                <cell>Status bar features</cell>
                <cell>src/status/</cell>
                <cell>Provider status indicators</cell>
            </row>
            <row>
                <cell>UI components</cell>
                <cell>src/ui/</cell>
                <cell>Settings pages, managers</cell>
            </row>
            <row>
                <cell>Core Copilot integration</cell>
                <cell>src/copilot/</cell>
                <cell>Completion providers, adapters</cell>
            </row>
            <row>
                <cell>Utility functions</cell>
                <cell>src/utils/</cell>
                <cell>Shared code across modules</cell>
            </row>
            <row>
                <cell>Type definitions</cell>
                <cell>src/types/</cell>
                <cell>VS Code and custom types</cell>
            </row>
            <row>
                <cell>AI prompts</cell>
                <cell>src/prompt/</cell>
                <cell>Instructions for different models</cell>
            </row>
        </table>
    </section>

    <section name="CODE_MAP">
        <note>No LSP available - project uses TypeScript without LSP server installed</note>
    </section>

    <section name="CONVENTIONS">
        <list>
            <item>Use 4-space indentation (except package.json: 2 spaces)</item>
            <item>Single quotes for strings</item>
            <item>No trailing commas</item>
            <item>Curly braces required</item>
            <item>Strict TypeScript mode enabled</item>
            <item>ES2022 target, Node16 modules</item>
            <item>Source maps enabled for debugging</item>
        </list>
    </section>

    <section name="ANTI_PATTERNS">
        <list>
            <item>Do not use deprecated VS Code APIs (see src/types/vscode.proposed.d.ts)</item>
            <item>Do not amend commits unless explicitly requested</item>
            <item>Do not use destructive git commands without approval</item>
            <item>Do not swallow errors in provider implementations</item>
            <item>Do not block on configuration retrieval</item>
            <item>Do not depend on chat-lib for commands in shim</item>
            <item>Do not use markdown code blocks in output</item>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OEvortex/aether](https://github.com/OEvortex/aether) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
