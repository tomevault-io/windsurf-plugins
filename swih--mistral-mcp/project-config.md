---
trigger: always_on
description: > Instructions de travail pour Claude (Opus 4.7) sur ce repo.
---

# CLAUDE.md — mistral-mcp

> Instructions de travail pour Claude (Opus 4.7) sur ce repo.
> Objectif : produire un MCP server Mistral **feature-complete** et **spec-compliant** qui peut dormir 6–12 mois sans régression.

---

## 1. Identité du projet

- **Nom** : `mistral-mcp` (npm + github.com/Swih/mistral-mcp)
- **Rôle** : serveur MCP stdio + Streamable HTTP wrappant l'API Mistral pour clients MCP (Claude Code, Cursor, Zed, Windsurf, Claude Desktop, ChatGPT Apps).
- **Versions en vigueur** :
  - MCP spec : **2025-11-25** (inclut structuredContent/outputSchema 2025-06-18, Streamable HTTP 2025-03-26, tool annotations).
  - SDK MCP : `@modelcontextprotocol/sdk@^1.29.0` — **toujours** via la high-level API `McpServer` + `registerTool/Resource/Prompt`. Ne jamais descendre au low-level `Server` sauf si explicitement requis par une feature spec (ex : sampling côté serveur).
  - SDK Mistral : `@mistralai/mistralai@^2.2.0` (Speakeasy-generated). Retry config obligatoire.
  - Node : `>=18`, TypeScript strict.

## 2. Règles dures (ne pas transgresser)

1. **Zéro claim non-vérifiable.** Si une feature n'est pas shippée dans ce repo et testée, elle n'existe pas dans le README, la doc, le CHANGELOG. Pas de "coming soon", pas de roadmap aspirationnelle vendue comme acquise.
2. **Langues : FR + EN uniquement.** Prompts, docs, commentaires, messages d'erreur. Aucune autre langue. Si on trouve de l'espagnol/allemand/etc. dans le code, on strippe.
3. **Spec-compliance >>> ergonomie.** Si la spec MCP dit "include `outputSchema`" ou "annotations are REQUIRED for read/write hints", on fait. Toujours.
4. **Chaque tool renvoie `content[]` ET `structuredContent`.** `content[]` = fallback lisible pour clients pré-2025-06-18. `structuredContent` = payload JSON strict conforme à `outputSchema`. Jamais l'un sans l'autre.
5. **Chaque tool déclare `annotations`** : `title`, `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`. Pour un wrapper API externe, `openWorldHint=true` quasi toujours.
6. **Erreurs API → `{ content: [text], isError: true }`**, jamais `throw`. Le LLM appelant doit pouvoir self-correct.
7. **Les inputs zod sont la source de vérité.** Jamais de cast `as any`, jamais de `z.any()`. Si un type Mistral n'est pas exporté, on le reconstruit avec zod, on ne "fait confiance".
8. **Le retry config est non-négociable** : `backoff` strategy, `initialInterval: 500`, `maxInterval: 5000`, `exponent: 2`, `maxElapsedTime: 30000`, `retryConnectionErrors: true`, `timeoutMs: 60000`.
9. **Pas d'env var autre que `MISTRAL_API_KEY`** sans justification. L'API key est lue UNE fois dans `src/index.ts`.
10. **Pas de dépendance runtime ajoutée** sans approbation explicite. Les seules deps runtime acceptées : `@modelcontextprotocol/sdk`, `@mistralai/mistralai`, `zod`. Point final.

## 3. Layout & responsabilités

```
src/
├── index.ts            # Entry point stdio. Bootstrap Mistral SDK + McpServer. Wiring uniquement.
├── models.ts           # Allow-lists Mistral (chat/embed/fim/tool/vision/audio/ocr). Zod enums. DEFAULT_*.
├── tools.ts            # Core chat/embed : mistral_chat (multimodal inclus), mistral_chat_stream, mistral_embed
├── tools-fn.ts         # Function calling + FIM : mistral_tool_call, codestral_fim
├── tools-vision.ts     # (v0.4) OCR : mistral_ocr
├── tools-audio.ts      # (v0.4) Voxtral : voxtral_transcribe, voxtral_speak
├── tools-agents.ts     # (v0.4) Agents + moderation : mistral_agent, mistral_moderate, mistral_classify
├── tools-files.ts      # (v0.4) Files API : files_upload/list/get/delete/signed_url
├── tools-batch.ts      # (v0.4) Batch API : batch_create/get/cancel/download
├── resources.ts        # mistral://models (LIVE call vers GET /v1/models, plus de liste figée)
├── prompts.ts          # Prompts curés FR (5+) + EN (optionnel)
├── transport.ts        # (v0.4) Streamable HTTP en plus de stdio. Sélection via CLI flag ou env.
└── shared.ts           # toTextBlock, errorResult, schemas communs (MessageSchema, UsageSchema, ImageContentSchema, ...)

test/
├── unit/               # vitest + InMemoryTransport + mocked SDK
├── stdio/              # spawn dist/index.js, StdioClientTransport e2e
├── live/               # MISTRAL_API_KEY requis, skipIf sinon
└── contract/           # Vérifie structuredContent === outputSchema pour chaque tool
```

## 4. Conventions de code

### Handler de tool (pattern canonique)

```ts
server.registerTool(
  "tool_name",
  {
    title: "...",
    description: "Quand l'utiliser / contraintes / format de retour. Pas de marketing.",
    inputSchema: { /* zod */ },
    outputSchema: { /* zod, payload strict */ },
    annotations: { title, readOnlyHint, destructiveHint, idempotentHint, openWorldHint },
  },
  async (input, extra) => {
    try {
      const res = await mistral.X.Y(input);
      const structured = mapToOutputSchema(res);
      return {
        content: [toTextBlock(summary(structured))],
        structuredContent: structured,
      };
    } catch (err) {
      return errorResult("tool_name", err);
    }
  }
);
```

### Règles zod
- Schemas composables dans `shared.ts` (ex: `MessageSchema`, `UsageSchema`, `ImageContentSchema`).
- Toujours `.describe()` sur les champs utilisateur — Claude et les autres LLMs lisent ces descriptions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Swih/mistral-mcp](https://github.com/Swih/mistral-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
