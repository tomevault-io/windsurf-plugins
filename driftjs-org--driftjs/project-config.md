---
trigger: always_on
description: DriftJS is a **register VM-based reactivity engine and AOT compiler** for high-performance browser UI and server-side rendering. It replaces traditional VDOM diffing and proxy-based reactivity with a lightweight, register-based bytecode stream executed by a virtual machine interpreter. Single File Components are authored in `.drift` files combining `<script>` logic and template markup (with `@if`, `@for`, `@switch` directives), compiled at build time into binary-like bytecode and constant pool e
---

# Agent Guide

## Project Overview

DriftJS is a **register VM-based reactivity engine and AOT compiler** for high-performance browser UI and server-side rendering. It replaces traditional VDOM diffing and proxy-based reactivity with a lightweight, register-based bytecode stream executed by a virtual machine interpreter. Single File Components are authored in `.drift` files combining `<script>` logic and template markup (with `@if`, `@for`, `@switch` directives), compiled at build time into binary-like bytecode and constant pool entries.

**Status:** Active development — modular monorepo.

**License:** MIT (Copyright 2026 Hrutav Modha)

---

## Repository Layout

```
driftjs/
├── packages/
│   ├── compiler/          # driftjs-compiler — Lexer, parser, transformer, generator
│   │   ├── index.ts       # Package entry: re-exports src/ and types/
│   │   ├── src/
│   │   │   ├── index.ts   # Barrel: re-exports lexer, parser, transformer, generator, compile()
│   │   │   ├── lexer.ts   # DriftLexer class
│   │   │   ├── parser.ts  # DriftParser class
│   │   │   ├── transformer.ts # DriftTransformer class (AST enrichment, @switch transformation)
│   │   │   └── generator.ts   # DriftGenerator class, astToJS() code emitter
│   │   ├── types/
│   │   │   ├── index.ts   # Barrel: re-exports all type modules
│   │   │   ├── ast.ts     # ASTNode, ElementNode, TextNode, InterpolationNode, IfNode, ForNode, SwitchNode
│   │   │   ├── token.ts   # TokenType, Token, SourceRange, SourceLocation
│   │   │   ├── opcodes.ts # Opcode enum, CompiledModule, ReactiveBinding, ImportSpec
│   │   │   ├── lexer-state.ts # DriftLexerState, LexerStateKind
│   │   │   └── error.ts   # DriftLexerError, DriftParserError
│   │   ├── tests/
│   │   │   ├── lexer.test.ts
│   │   │   ├── parser.test.ts
│   │   │   ├── transformer.test.ts
│   │   │   └── generator.test.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── dom/               # driftjs-dom — Browser Client Register VM & Reconciler
│   │   ├── index.ts       # Package entry: DriftClientVM, mount(), hydrate(), Context API
│   │   ├── src/
│   │   │   ├── index.ts   # DriftClientVM class, execution loop, reactive regions, event delegation
│   │   │   ├── reconciler.ts # Keyed list LIS reconciler (reconcileKeyedList)
│   │   │   └── hydration.ts  # TreeWalker-based HydrationCursor
│   │   ├── types/
│   │   │   └── index.ts   # VMExecutionOptions, ReactiveRegion, LoopFrame, ItemRecord
│   │   ├── tests/
│   │   │   ├── client.test.ts
│   │   │   ├── derived.test.ts
│   │   │   ├── edge-cases.test.ts
│   │   │   ├── context.test.ts
│   │   │   ├── async.test.ts
│   │   │   ├── if.test.ts
│   │   │   ├── for.test.ts
│   │   │   ├── switch.test.ts
│   │   │   └── hydration.test.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── ssr/               # driftjs-ssr — Headless Server VM & HTML Serializer
│   │   ├── index.ts       # Package entry: DriftServerVM, renderToString(), serializeNode()
│   │   ├── src/
│   │   │   └── index.ts   # DriftServerVM execution, virtual node tree builder, HTML serializer
│   │   ├── types/
│   │   │   └── index.ts   # SSRExecutionOptions, ServerNode, ServerElementNode, ServerTextNode
│   │   ├── tests/
│   │   │   ├── ssr.test.ts
│   │   │   └── context.test.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── utils/             # driftjs-shared — Shared Scope, Context, & Evaluators
│   │   ├── index.ts       # Package entry: scope helpers, evaluator, context, constants
│   │   ├── src/
│   │   │   ├── index.ts   # Barrel export
│   │   │   ├── constants.ts   # MAX_REGISTERS (256)
│   │   │   ├── scope.ts       # setScopeValue(), inScopeChain()
│   │   │   ├── evaluator.ts   # evaluateExpression(), evaluatePropsSpec(), resolveIterable()
│   │   │   └── context.ts     # createContext(), provideContext(), injectContext(), pushActiveVM()
│   │   ├── types/
│   │   │   └── index.ts   # BaseVMExecutionOptions, Context<T>
│   │   ├── tests/
│   │   │   └── utils.test.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── router/            # driftjs-router — Client-Side SPA Router
│   │   ├── index.ts       # Package entry: re-exports src/ and types/
│   │   ├── src/           # Implementation logic: router, matcher, history, components
│   │   │   ├── index.ts   # Barrel: re-exports router, matcher, history, components
│   │   │   ├── router.ts  # createRouter(), RouterContext, navigation engine
│   │   │   ├── matcher.ts # Route pattern matching, parameter and query extraction
│   │   │   ├── history.ts # BrowserHistory, HashHistory, MemoryHistory drivers
│   │   │   └── components.ts # RouterView and Link native component helpers
│   │   ├── types/         # Type definitions: RouteRecord, MatchedRoute, Router, RouterOptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [driftjs-org/driftjs](https://github.com/driftjs-org/driftjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
