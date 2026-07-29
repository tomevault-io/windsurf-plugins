---
trigger: always_on
description: validateSession,
---

# Conventions

Patterns enforced across this codebase. CI catches the mechanical ones (lint, typecheck, format); the rest live in PR review.

If you're unsure about a non-mechanical rule, find the closest existing file and follow its shape — symmetry across the codebase is a feature.

## TypeScript

- `strict: true` and `noUncheckedIndexedAccess: true` are non-negotiable.
- No `any` in source files. If a library returns `unknown`, narrow with a type predicate or a Zod schema; don't pass it through.
- No `@ts-ignore` without a same-line comment that names the underlying bug or upstream issue. Prefer `@ts-expect-error` so the suppression becomes a CI failure once the upstream is fixed.
- Imports use the `@/...` aliases declared in `tsconfig.json`. Never use deep relative imports (`../../../`).
- Prefer `import type { … }` for type-only imports; bundlers and `verbatimModuleSyntax` strip them automatically.

## File organization

Every source file has exactly one of the following responsibilities. If a new file doesn't fit, the convention is wrong, not the file — open a PR to extend this list.

| Directory                     | What lives here                                                               | Imports from           |
| ----------------------------- | ----------------------------------------------------------------------------- | ---------------------- |
| `prisma/services/`            | Database access functions. Only place that imports `@prisma/client` directly. | server only            |
| `src/lib/server/`             | Server-only utilities (auth, mail, Stripe, raw body, authorization).          | server only            |
| `src/lib/common/`             | Code safe to import from either side (e.g. the `apiFetch` helper).            | either                 |
| `src/lib/client/`             | Browser-only utilities (Clipboard helper, Stripe.js).                         | client only            |
| `src/config/api-validation/`  | One Zod schema per endpoint, re-exported from `index.ts`.                     | either                 |
| `src/config/email-templates/` | `html()` + `text()` generators, one file per email.                           | server only            |
| `src/components/`             | Presentational React components. No data fetching.                            | client                 |
| `src/sections/`               | Landing-page composition (Hero, Features, Pricing, …).                        | client                 |
| `src/layouts/`                | Page chrome: `AccountLayout`, `AuthLayout`, `LandingLayout`, `PublicLayout`.  | client                 |
| `src/hooks/data/`             | One `useSWR` wrapper per resource.                                            | client                 |
| `src/providers/`              | React Context providers.                                                      | client                 |
| `src/pages/`                  | Routes (Pages Router).                                                        | either, route by route |
| `src/types/`                  | Ambient `.d.ts` files (module augmentation only).                             | n/a                    |

The server / client boundary is a hard rule: importing `src/lib/server/*` from anything under `src/components/`, `src/sections/`, or a client-only branch of `src/pages/` will leak server secrets into the bundle. Treat it like a type error.

## API routes

Every API route follows the same skeleton. The canonical example is `src/pages/api/workspace/[workspaceSlug]/name.ts`:

```ts
import type { NextApiRequest, NextApiResponse } from 'next';

import {
  updateWorkspaceNameSchema,
  validateSession,
} from '@/config/api-validation';
import { requireWorkspaceOwner } from '@/lib/server/authorization';
import { parseBody } from '@/lib/server/validate';
import { updateName } from '@/prisma/services/workspace';

const handler = async (req: NextApiRequest, res: NextApiResponse) => {
  const { method } = req;

  if (method === 'PUT') {
    const session = await validateSession(req, res);
    const body = parseBody(updateWorkspaceNameSchema, req.body, res);
    if (!body) return;
    const { workspaceSlug } = req.query as { workspaceSlug: string };
    const workspace = await requireWorkspaceOwner(
      req,
      res,
      session,
      workspaceSlug
    );
    if (!workspace) return;
    updateName(
      session.user.userId,
      session.user.email,
      body.name,
      workspaceSlug
    )
      .then((name) => res.status(200).json({ data: { name } }))
      .catch((error: Error) =>
        res.status(404).json({ errors: { error: { msg: error.message } } })
      );
  } else {
    res
      .status(405)
      .json({ errors: { error: { msg: `${method} method unsupported` } } });
  }
};

export default handler;
```

Notes:

- The method switch is a single `if / else if / else` chain. We don't use a framework like next-connect.
- `validateSession` ALWAYS comes first.
- `parseBody` comes next on routes that take a request body.
- The authorization helper comes next on routes that touch workspace data.
- Services return the new state; the route serialises it to `{ data: … }`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nextacular/nextacular](https://github.com/nextacular/nextacular) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
