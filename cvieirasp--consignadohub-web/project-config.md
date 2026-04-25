---
trigger: always_on
description: This document defines architecture, conventions, patterns, and expectations for the `consignadohub-web` project.
---

# ConsignadoHub Frontend — Angular 21 Project Context

This document defines architecture, conventions, patterns, and expectations for the `consignadohub-web` project.

---

## 1) Project Summary

**consignadohub-web** is the Angular 21 frontend for ConsignadoHub. It consumes the backend APIs (CustomerService, ProposalService), authenticates via Keycloak OIDC, and provides a production-grade UI for the consigned credit proposal pipeline.

**Key technical choices:**

- **Angular 21** — standalone components, Signals, new control-flow syntax (`@if`, `@for`, `@defer`)
- **Angular Signals** — primary reactive primitive; RxJS used only for async streams that warrant it
- **Keycloak JS** — OIDC authentication (no third-party Angular wrapper)
- **Functional interceptors + guards** — no class-based boilerplate
- **SCSS** — component-scoped styles
- **Vitest + Angular Testing Library** — unit and component tests

---

## 2) Backend Recap (Contracts the Frontend Must Respect)

| Concern | Rule |
|---|---|
| API versioning | All endpoints under `/v1/...` |
| Error format | RFC 7807 **ProblemDetails** with `extensions.correlationId`, `extensions.errorCode`, `extensions.validationErrors` |
| Auth | Keycloak OIDC; JWT Bearer required on all non-simulate endpoints |
| Correlation ID | Frontend generates UUID per request; sent as `X-Correlation-Id` header |
| CPF format | Backend normalizes to unformatted digits (e.g., `98765432100`) |
| Proposal statuses | `Submitted`, `UnderAnalysis`, `Approved`, `Rejected`, `ContractGenerated`, `Disbursed` |

---

## 3) Tech Stack & Dependencies

```bash
# Create project
ng new consignadohub-web --routing --style=scss --ssr=false
# Auth
npm install keycloak-js
# Correlation ID generation
npm install uuid
npm install --save-dev @types/uuid

```

### Angular Version

- Angular CLI 21 / Angular 21
- Node.js 22 LTS (minimum 20)
- TypeScript strict mode enabled

---

## 4) Repository Layout

```
consignadohub-web/
 src/
   app/
     core/
       auth/
         keycloak.service.ts        # Keycloak init, token, roles
         auth.guard.ts              # Route guard (authenticated)
         role.guard.ts              # Route guard (role-based)
       interceptors/
         auth.interceptor.ts        # Attaches JWT + X-Correlation-Id
         error.interceptor.ts       # Normalizes ProblemDetails errors
       services/
         customer.service.ts
         proposal.service.ts
       models/
         problem-details.model.ts
     shared/
       components/                  # Toast, Spinner, ErrorMessage, ConfirmDialog
       pipes/                       # CpfPipe, CurrencyBrlPipe, ProposalStatusPipe
       directives/
     features/
       customers/
         list/
         create/
         detail/
       proposals/
         simulate/
         submit/
         detail/                    # Status + timeline view
       dashboard/
     app.routes.ts
     app.config.ts
     app.component.ts
   environments/
     environment.ts
     environment.prod.ts
 .github/
   workflows/
     ci.yml
```

---

## 5) Environment Configuration

`src/environments/environment.ts`

```ts
export const environment = {
 production: false,
 apiBaseUrl: 'http://localhost:5100/v1', // CustomerService
 proposalApiBaseUrl: 'http://localhost:5200/v1', // ProposalService
 keycloak: {
   url: 'http://localhost:8080',
   realm: 'consignadohub',
   clientId: 'consignadohub-web' // public PKCE client
 }
};
```

`src/environments/environment.prod.ts`

```ts
export const environment = {
 production: true,
 apiBaseUrl: 'https://api.consignadohub.example.com/v1',
 proposalApiBaseUrl: 'https://proposals.consignadohub.example.com/v1',
 keycloak: {
   url: 'https://auth.consignadohub.example.com',
   realm: 'consignadohub',
   clientId: 'consignadohub-web'
 }
};
```

---

## 6) Core Models

### ProblemDetails

`src/app/core/models/problem-details.model.ts`

```ts
export interface ProblemDetails {
 type?: string;
 title?: string;
 status?: number;
 detail?: string;
 instance?: string;
 extensions?: {
   correlationId?: string;
   errorCode?: string;
   validationErrors?: Record<string, string[]>;
 };
}
```

### Customer

`src/app/core/models/customer.model.ts`

```ts
export interface Customer {
 id: string;
 name: string;
 cpf: string;         // normalized digits only (no dots/dashes)
 email: string;
 phone?: string;
 isActive: boolean;
 createdAt: string;   // ISO 8601 UTC
}

export interface CreateCustomerRequest {
 name: string;
 cpf: string;
 email: string;
 phone?: string;
}

export interface UpdateCustomerRequest {
 name: string;
 email: string;
 phone?: string;
}

export interface CustomerSearchParams {
 name?: string;
 cpf?: string;
 page?: number;
 pageSize?: number;
}
```
### Proposal

`src/app/core/models/proposal.model.ts`

```ts
export type ProposalStatus =
 | 'Submitted'
 | 'UnderAnalysis'
 | 'Approved'
 | 'Rejected'
 | 'ContractGenerated'
 | 'Disbursed';

export interface Proposal {
 id: string;
 customerId: string;
 requestedAmount: number;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cvieirasp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
