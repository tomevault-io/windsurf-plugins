---
trigger: always_on
description: >-
---


# Engineering Patterns for SvelteKit

This project runs in SPA mode (`ssr = false`). All code executes in the browser — there are no server routes, server hooks, or server-side rendering.

SPA implications for pattern selection:
- Module-level singletons are safe (one user per tab, no cross-request leakage).
- Strategies wrap client-side SDKs or call external APIs/edge functions.
- Observer patterns use Svelte 5 runes; event buses are client-side only.
- Services and repositories run entirely in the browser via the Supabase client SDK.
- If the project ever migrates to SSR, re-evaluate Singleton and Observer patterns for per-request safety.

Composition flow — patterns layer in one direction:

```
Component → Service → Repository → Adapter (wraps SDK)
                                  ↑
                            Singleton (shared client)
```

Components call services. Services enforce business rules and call repositories. Repositories abstract data access. Adapters wrap third-party SDKs. Singletons provide shared client instances. Strategies and Factories are cross-cutting — used wherever swappable behavior or complex construction is needed.

============================================================
1) Factory — centralized object creation
============================================================
Use a factory when constructing objects requires conditional logic, defaults, or async setup that callers should not repeat.

- Export a plain function (or async function) that returns a fully configured instance.
- Keep construction details hidden; callers receive a typed result.
- Prefer a factory over a class constructor when multiple creation paths exist.
- For costly async setup (remote config, auth token exchange), await the factory once at init time and reuse the result.

```typescript
// src/lib/api/create-api-client.ts
import type { ApiClient } from './types';

// Async factory — resolves auth headers before returning a ready client.
export async function createApiClient(baseUrl: string): Promise<ApiClient> {
  const token = await fetchServiceToken();
  const headers = { Authorization: `Bearer ${token}` };

  return {
    get: async (path) => fetch(`${baseUrl}${path}`, { headers }).then((r) => r.json()),
    post: async (path, body) =>
      fetch(`${baseUrl}${path}`, {
        method: 'POST',
        headers: { ...headers, 'Content-Type': 'application/json' },
        body: JSON.stringify(body),
      }).then((r) => r.json()),
  };
}
```

Avoid:
- Scattering construction logic across multiple call sites — centralize it in the factory.
- Returning partially initialized objects — the factory's output must be ready to use.

============================================================
2) Repository — data access abstraction
============================================================
Encapsulate all Supabase table operations behind a repository interface so business logic never depends on Supabase directly.

- Define an interface describing the data operations (find, create, update, delete).
- Implement with Supabase queries; export the concrete implementation as the default.
- For tests, provide a mock factory that satisfies the same interface with in-memory storage.
- Place repositories in `src/lib/database/repositories/`.
- Select only the columns you need — avoid `select('*')`.
- Let Supabase errors propagate — the service layer is responsible for catching and transforming them.

```typescript
// src/lib/database/repositories/types.ts
export interface ProjectRepository {
  findById(id: string): Promise<Project | null>;
  findByOwner(ownerId: string): Promise<Project[]>;
  create(data: CreateProjectInput): Promise<Project>;
  update(id: string, data: UpdateProjectInput): Promise<Project>;
  remove(id: string): Promise<void>;
}
```

```typescript
// src/lib/database/repositories/project-repository.ts
import { supabase } from '$lib/supabase/client';
import type { ProjectRepository } from './types';

export const projectRepository: ProjectRepository = {
  async findById(id) {
    const { data, error } = await supabase
      .from('projects')
      .select('id, name, owner_id, created_at')
      .eq('id', id)
      .maybeSingle();
    if (error) throw error;
    return data;
  },
  // ... remaining methods follow the same shape
};
```

```typescript
// src/lib/database/repositories/project-repository.mock.ts
import type { ProjectRepository } from './types';

// In-memory mock for unit tests — no Supabase dependency.
export function createMockProjectRepository(
  seed: Project[] = []
): ProjectRepository {
  const store = new Map(seed.map((p) => [p.id, p]));

  return {
    async findById(id) {
      return store.get(id) ?? null;
    },
    async findByOwner(ownerId) {
      return [...store.values()].filter((p) => p.owner_id === ownerId);
    },
    async create(data) {
      const project = {
        ...data,
        id: crypto.randomUUID(),
        created_at: new Date().toISOString(),
      } as Project;
      store.set(project.id, project);
      return project;
    },
    async update(id, data) {
      const existing = store.get(id);
      if (!existing) throw new Error('Not found');
      const updated = { ...existing, ...data };
      store.set(id, updated);
      return updated;
    },
    async remove(id) {
      store.delete(id);
    },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wesselgrift/sveltekit-spa](https://github.com/wesselgrift/sveltekit-spa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
