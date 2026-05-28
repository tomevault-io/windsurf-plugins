---
trigger: always_on
description: All rules in this document should be treated as strict coding and architectural requirements for this repository.
---


# Instruction

All rules in this document should be treated as strict coding and architectural requirements for this repository.  
When generating or editing code, always choose the option that is consistent with these rules.  
When reviewing code, Copilot must also evaluate changes according to these rules and suggest corrections whenever the submitted code violates any of them.

# Repository Overview

- **Rule:** Vera is a video call application built with Vonage Video API and is being transformed into a reusable component library for Vonage Video API React SDK users.
- **Rule:** Video components, hooks, and primitives must be extracted into `libs/ui` and `libs/core`.
- **Rule:** Backend-agnostic Vonage Video API handler logic must live in `libs/api`.
- **Rule:** New features must be designed with **reusability** in mind and must remain **agnostic of Vera** wherever possible.

---

# Project Architecture & Libraries

This is a mono repo containing:

- Libs: `ui`, `core`, `common`, `api`
- Main projects: `frontend`, `backend`
- `integration-test`

---

# Technology Stack

- Frontend: React, TypeScript, MUI, react-router-dom, axios  
- Backend: Node.js, TypeScript, Express  
- Integration tests: Playwright, TypeScript  
- libs/ui: React, TypeScript, MUI  
- libs/core: React, TypeScript  
- libs/common: React, TypeScript  

React version: `^19.2`  
TypeScript version: `^5.8.3`

---

# General Development Guidelines, Import Rules

## Library and feature placement

- **Rule:** If a component is completely stateless and generic, it must be placed in:
  - `libs/ui` for visual components  
  - `libs/core` if it is faceless (non-visual logic).
  - `libs/common` for helpers, utilities, and hooks that are agnostic of the project.
- **Rule:** Vera-specific business logic (roles, permissions, product policy/decisions) must stay in the app layer (`frontend`/`backend`).
- **Rule:** This is **especially enforced** for video-related components such as publishers, subscribers, sessions, `videoView`s, etc.
- **Rule:** Helpers, utilities, and hooks that are agnostic of the project must be placed in `libs/common`.
- **Rule:** Logic that is shared between different projects (**frontend**, **backend**, **libs**) should be proposed for `libs/common`.
- **Rule:** Do not add new dependencies. Use only the installed dependencies.
- **Rule:** Do not add new state management libraries. Use only existing tooling.
- **Rule:** Components must be kept small, focused, and composable.

## Import rules

- **Rule:** Always prefer specific imports over deep namespace imports.

**Violation:**

```tsx
// Bad
import { isNil } from 'lodash';
```

**Correct:**

```tsx
// Good
import isNil from 'lodash/isNil';
```

---

# Coding Style & Programming Patterns

- **Rule:** Prefer early returns / fast-fail to reduce nesting.
- **Rule:** Prefer **linear code** whenever possible.
- **Rule:** Prefer IIFE (Immediately Invoked Function Expression) for computing values linearly instead of branching assignments.

**Violation:**

```tsx
// Bad
let url = '';

if (condition1) {
    url = 'value1';
} else {
    if (condition2) {
        url = 'value2';
    } else {
        url = 'value3';
    }
}

this.API_URL = url;
```

**Correct:**

```tsx
// Good
const url = (() => {
    if (condition1) return 'value1';
    if (condition2) return 'value2';
    return 'value3';
})();

this.API_URL = url;
```

- **Rule:** Use named boolean expressions or named helper functions for complex boolean conditions.

**Violation:**

```tsx
// Bad
if (user.isAdmin && user.isActive && hasValidSubscription(user)) {
    // ...
}
```

**Correct:**

```tsx
// Good
const isUserEligible =
    user.isAdmin && user.isActive && hasValidSubscription(user);

if (isUserEligible) {
    // ...
}
```

**Correct (extracted helper):**

```tsx
// Good
if (isUserEligible(user)) {
    // ...
}
```

**Correct (simple null check):**

```tsx
// Good
if (isNil(data)) return;
```

- **Rule:** Acronyms in names are banned across the codebase, except `req` and `res` when working with Express `Request` and `Response`.
- **Rule:** Use fully descriptive names, even if they are longer. Minification handles bundle size.

**Violation:**

```tsx
// Bad
function fetchUsrDtls() {
    // ...
}
```

**Correct:**

```tsx
// Good
function fetchUserDetails() {
    // ...
}
```

**Violation:**

```tsx
// Bad
const vc = new VideoClient();
```

**Correct:**

```tsx
// Good
const videoClient = new VonageVideoClient();
```

- **Rule:** Prefer linear `tryCatch` helpers instead of nested `try/catch`.
- **Rule:** Nested `try/catch` blocks are banned.

**Violation:**

```tsx
// Bad
const ValidateRequest = async (req) => {
    const validator = new RequestValidator();

    let error = null;
    let data = null;
    
    // nested try/catch just make the code messy and hard to read
    try {
        data = parseRequest(req.body);
    } catch (error) {
        error = error;
    }

    if(error || !data) {
        validator.addError(error?.message || 'Unknown parsing error');
    }

    // this is okay
    validator.assert();
};
```

**Correct:**

```tsx
// Good
import tryCatch from '@common/execution/tryCatch';

const myFunction = async () => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vonage/vonage-video-react-app](https://github.com/Vonage/vonage-video-react-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
