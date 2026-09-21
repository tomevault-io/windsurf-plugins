---
trigger: always_on
description: Astro 7 with React 18 islands, TypeScript. Notes for working in this repo.
---

# tokendashboard-frontend

Astro 7 with React 18 islands, TypeScript. Notes for working in this repo.

## Imports

Anything outside the current directory is imported through the `~/` alias, which maps
to `src/`. Siblings stay relative:

```ts
import { usdCentToUsd } from '~/lib/types';
import { dict, type Locale } from '~/i18n';
import KpiCard from './KpiCard';
```

`../lib/api` and `../../lib/api` are rejected by
`@typescript-eslint/no-restricted-imports` — `./sibling` is not, because an alias buys
nothing there and only lengthens the specifier.

The mapping is declared once, in `tsconfig.json` `compilerOptions.paths` — deliberately
without `baseUrl`, which would additionally make `src/i18n` resolve as a bare specifier
and open a second import form the ESLint rule cannot see. Astro reads the paths and
hands them to Vite, so the same alias resolves under `astro dev`, `astro build`,
`astro check` and vitest; there is no second copy in `vitest.config.ts` to keep in sync.
`~` rather than `@`, because `@` is already the npm scope prefix (`@astrojs/react`,
`@testing-library/react`).

It applies to every specifier, not just `import`: `vi.mock('~/lib/api')` still finds
`src/lib/__mocks__/api.ts`, and a `__mocks__` file reaching for the real module uses
`vi.importActual('~/lib/api')` — the same path it is registered under, which resolves to
the real module rather than recursing.

## Tests

### Structure: given / when / then

Every test body is divided by `// given`, `// when` and `// then` comments, in that
order. All three are mandatory — not decoration. If one of the phases has nothing to
put under it, that is a signal about the test, not a licence to drop the comment: a
test with no `// when` is asserting on a constant, and one with no `// given` usually
depends on hidden ambient state.

```ts
it('normalises a reversed range', () => {
  // given
  const reversed = { from: '2026-03-31', to: '2026-03-01' };

  // when
  const result = normalize(reversed);

  // then
  expect(result).toEqual({ from: '2026-03-01', to: '2026-03-31' });
});
```

Keep the phases in that order and put the assertions last. Setup that belongs to the
whole `describe` block goes into `beforeEach` rather than being repeated in every
test — the `// given` then stays, naming where the setup actually happened:

```ts
it('loadSelection defaults when nothing is set', () => {
  // given — beforeEach cleared both the URL params and localStorage

  // when
  const selection = loadSelection();

  // then
  expect(selection).toEqual(DEFAULT_SELECTION);
});
```

A rejection is the one case where `when` and `then` genuinely cannot be separated,
because the call *is* the subject of the assertion. Combine those two and say so:

```ts
// given
mockFetch({}, false, 500);

// when / then — a rejection cannot be split; the call is the assertion subject
await expect(fetchProviderUsage('claude', range)).rejects.toThrow('500');
```

### Exception: table-style checks

When a test body is nothing but one-line assertions and *each line is already a
complete given + when + then* — a lookup table, scale boundaries, the same call per
locale — do not split it artificially. Collapse the three phases into one combined
comment above the block. This covers a single such assertion as well as a series of
them; one line is the most artificial case of all to split.

```ts
it('de: Tsd./Mio./Mrd.', () => {
  // given / when / then — one scale boundary per line
  expect(formatTokens(1_000, 'de')).toEqual({ value: '1,0', unit: 'Tsd.' });
  expect(formatTokens(1_000_000, 'de')).toEqual({ value: '1,0', unit: 'Mio.' });
  expect(formatTokens(1_000_000_000, 'de')).toEqual({ value: '1,0', unit: 'Mrd.' });
});
```

Breaking each of those lines into three phases would triple the line count without
adding information.

The operative test when deciding: **does the assertion fit on one line with the call
inline?** If yes, collapse. If the test needs its own setup statements, or the
assertion has to span several lines, split it — and extract the call into a named
variable under `// when`, which is what makes the split worth having:

```ts
// before
expect(eachDay({ from: '2026-07-01', to: '2026-07-03' })).toEqual([
  '2026-07-01',
  '2026-07-02',
  '2026-07-03',
]);

// after
// given
const range = { from: '2026-07-01', to: '2026-07-03' };

// when
const days = eachDay(range);

// then
expect(days).toEqual(['2026-07-01', '2026-07-02', '2026-07-03']);
```

`src/lib/format.test.ts` is the reference for the fully collapsed style;
`src/lib/range.test.ts` mixes both and is the reference for the judgement call.

### Running them

| Command | Purpose |
| --- | --- |
| `npm test` | Whole suite once (`vitest run`) — this is what CI runs |
| `npx vitest` | Watch mode |
| `npx vitest run src/lib/format.test.ts` | A single file |
| `npx vitest run -t 'thousands separators'` | Only tests whose name matches |

`vitest.config.ts` builds on Astro's own Vite config via `getViteConfig`, so imports
resolve exactly as they do under `astro dev` and `astro build`. The environment is
`jsdom`, and `src/**/*.test.{ts,tsx}` is collected — a component test can be written
in `.tsx` with plain JSX.

### Component tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neuland/tokendashboard-frontend](https://github.com/neuland/tokendashboard-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
