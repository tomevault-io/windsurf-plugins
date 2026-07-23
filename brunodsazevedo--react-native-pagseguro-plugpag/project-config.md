---
trigger: always_on
description: Biblioteca React Native (TurboModule / New Architecture) que expõe o SDK
---

# react-native-pagseguro-plugpag

## Visão Geral do Projeto

Biblioteca React Native (TurboModule / New Architecture) que expõe o SDK
`PlugPagServiceWrapper` da PagSeguro para aplicativos React Native. Foco exclusivo em
Android — terminals PagBank SmartPOS (A920, A930, P2, S920). iOS é explicitamente fora
de escopo (guarda de dois níveis em `src/index.ts`).

**Package**: `react-native-pagseguro-plugpag` | **Versão**: 0.1.0
**SDK Alvo**: `br.com.uol.pagseguro.plugpagservice.wrapper:wrapper:1.35.0`
**Maven**: `https://github.com/pagseguro/PlugPagServiceWrapper/raw/master`
**Mínimo Android SDK**: 24 | **Compile/Target SDK**: 36

---

## Stack de Tecnologias

| Camada | Tecnologia |
|---|---|
| JS / Spec | TypeScript 5.9 (`strict: true`) |
| Native Android | Kotlin 2.0.21 |
| Framework | React Native 0.83.2 (New Architecture) |
| Build | react-native-builder-bob 0.40.18 |
| Expo Plugin | @expo/config-plugins ^9.0.0 |
| Threading nativo | kotlinx.coroutines (`Dispatchers.IO` p/ métodos bloqueantes) + `UiThreadUtil` (p/ callbacks async/RxJava) |
| Testes JS | Jest 29 + react-native preset |
| Testes Kotlin | JUnit 5 + Mockk |
| Lint | ESLint (flat config) + Prettier |
| Package manager | Yarn 4.11.0 |

---

## Estrutura de Arquivos Críticos

```
src/
├── NativePagseguroPlugpag.ts        ← Spec TurboModule
├── functions/
│   ├── activation/
│   │   ├── types.ts
│   │   └── index.ts
│   ├── payment/
│   │   ├── types.ts
│   │   └── index.ts
│   ├── refund/
│   │   ├── types.ts
│   │   └── index.ts
│   ├── print/
│   │   ├── types.ts
│   │   └── index.ts
│   └── index.ts
├── hooks/
│   └── usePaymentProgress.ts
├── types/
│   └── sharedTypes.ts               ← PlugPagTransactionResult (compartilhado por payment e refund)
└── index.ts                         ← barrel raiz (iOS guard Nível 1 + re-exports)

src/__tests__/
├── functions/
│   ├── activation.test.ts
│   ├── payment.test.ts
│   ├── refund.test.ts
│   └── print.test.ts
├── hooks/
│   └── usePaymentProgress.test.ts
└── index.test.ts                    ← apenas iOS guard Nível 1

android/src/main/java/com/pagseguroplugpag/
├── PagseguroPlugpagModule.kt   ← Implementação Kotlin do TurboModule
└── PagseguroPlugpagPackage.kt  ← Registro do módulo React Native

android/build.gradle            ← Configuração Android + dependência SDK PagBank

specs/                          ← Documentação especkit por feature
├── 001-pagseguro-sdk-setup/
└── 002-pinpad-activation/

example/src/App.tsx             ← App de demonstração
```

---

## Constituição do Projeto (v1.3.0)

A constituição completa está em `.specify/memory/constitution.md`. Os princípios são
**NON-NEGOTIABLE** exceto quando explicitamente justificado.

### Princípio I — TurboModules Only

- Toda comunicação JS↔Native DEVE usar JSI/TurboModule. Bridge legada é **proibida**.
- `NativePagseguroPlugpag.ts` é a **única fonte de verdade** do contrato JS↔Native.
- `.podspec` e `ios/` NÃO DEVEM existir no repositório.
- React Native mínimo: 0.76.

### Princípio II — TypeScript Strict — Zero `any`

- `tsconfig.json`: `strict: true`, `noUnusedLocals`, `noUnusedParameters`,
  `noUncheckedIndexedAccess`, `verbatimModuleSyntax`.
- `any` é **proibido**. Toda exceção DEVE ser documentada com `// EXCEPTION: <razão>`.
- `@ts-ignore` / `@ts-expect-error` são **proibidos** sem justificativa documentada.
- Tipos complexos usam `Object` na spec do codegen (exigência do codegen — permitido)
  e são tipados com type assertion segura na camada pública.
- Enums DEVEM ser `const` objects (não `enum` nativo do TypeScript) — tree-shakeable.
- Interfaces DEVEM ser usadas para todos os modelos de dados; `object` genérico é proibido.

### Princípio III — Test-First / TDD

- Testes DEVEM ser escritos **antes** da implementação e confirmados como falhando.
- 100% das funções exportadas de `src/index.ts` DEVEM ter cobertura de teste unitário.
- Todo novo método nativo DEVE ter teste de integração Kotlin (JUnit 5 + Mockk).
- O módulo nativo (`NativePagseguroPlugpag`) DEVE sempre ser mockado em testes unitários JS.
- PR que quebra teste existente é **bloqueado**.

### Princípio IV — Clean Code + SOLID

- Cada módulo TypeScript possui um único domínio (`payment`, `print`, `nfc`, `activation`).
- `PlugPag` (SDK) DEVE ser instanciado e chamado **somente** dentro de
  `PagseguroPlugpagModule.kt`. Nenhuma lógica de negócio além de serialização e chamadas
  SDK é permitida no módulo Kotlin.
- Tipos DEVEM ser estendidos via union types; contratos existentes NÃO DEVEM ser quebrados.
- O módulo nativo DEVE ser acessado sempre via interface Spec, nunca diretamente.

### Princípio V — Device Compatibility & Fail-Fast

- A biblioteca DEVE detectar se está rodando em terminal PagBank SmartPOS.
- **Dispositivo POS**: SDK roda normalmente.
- **Não-POS + `__DEV__ = true`**: warning + respostas mock (cobre toda a API surface).
- **Não-POS + produção**: qualquer chamada DEVE lançar erro explícito. Fallback silencioso
  é **proibido**.
- **Status**: ⚠️ DEFERRED — não implementado ainda; será feature dedicada.

### Princípio VI — Android-Only Scope

- Todo código nativo DEVE ser Kotlin 2.x.
- Threading para chamadas SDK DEVE seguir a **Threading Policy** (Constituição v1.4.0,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brunodsazevedo/react-native-pagseguro-plugpag](https://github.com/brunodsazevedo/react-native-pagseguro-plugpag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
