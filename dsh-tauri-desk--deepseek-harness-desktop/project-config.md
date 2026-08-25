---
trigger: always_on
description: DeepSeek Harness desktop (Tauri 2 + React 18), embeds the Harness UI served at `http://127.0.0.1:3080`.
---

# Development Specification Document

DeepSeek Harness desktop (Tauri 2 + React 18), embeds the Harness UI served at `http://127.0.0.1:3080`.

- **端口隔离**：release 默认 `3080`，debug（`pnpm tauri dev` / `cargo build`）默认 `3081`，由 `config::setting::default_port()` 用 `cfg!(debug_assertions)` 区分，避免开发时与已运行的桌面端争用端口。
- **数据隔离（核心共用、数据不共用）**：node/`dependencies/dsh`/`dependencies/pnpm` 为共用核心（AppData）；debug 构建的 `$DSH_HOME` 默认为 `~/.dsh.dev`（`config::runtime::get_dsh_data_path` 用 `cfg!(debug_assertions)` 区分）且 store 用独立文件 `.store.dev.dat`（`config::setting::store_dat_file_name`），避免开发版与生产版会话/档案/端口状态互相污染，也防止 dev 版热重启把 release 的服务进程杀掉（`service/workflow::terminate_stale_harness_processes` 在 debug 下为 no-op，改由 `.dsh.dev/.harness.pid` 精确回收）。debug 构建不迁移旧数据（`service/migrate`）、不注册/注销 PATH、不写烘焙 DSH_HOME 的 `dsh` shim（`service/cli`）。
- **Windows 极简模式**：预装插件流程（`service/plugin`）对 Windows 用户列出「修复」项（`dsh-win-terminal-inspector`，黄色 chip 默认勾选），确认后 `dsh plugin add github:clearkurt/dsh-win-terminal-inspector` 从 GitHub 安装（桌面端**不内置**插件源码）；随后 `service/workflow/win_inspector.rs`（仅 Windows，幂等）写入 profile `cordis.patch.yml` 挂载行并创作 `$DSH_HOME/.agent-presets/minimal-win/` 用户 preset（Git Bash + danger-full-access，因为 agent preset 组成不受 profile patch 管辖）。

- Prioritize using customized components from src/components, hero-ui.
- This will help minimize the need for writing custom classes.
- If you write new content, you need to handle i18n en keys
- i18n keys must be flat (no nesting), use dot-notation flat keys only
- No hardcoded strings; sync `src/i18n/zh.ts`, `en.ts` and `types.ts`
- If the component you write/modify is too complex, you need to split it into multiple components
- Repeated logic should be encapsulated into methods/components

## Tech Stack

- **Frontend**: React 18 + TS + Tailwind 4 (no plain CSS), Vite (`src/`)
- **Backend**: Rust / Tauri 2 (`src-tauri/src/`)
  - `bridge/cmd.rs`: Tauri commands (register in `lib.rs` `generate_handler!`)
  - `config/`: constants, paths (`runtime.rs`), settings (`setting.rs`), i18n & theme
  - `service/download/`: Node/Dsh/pnpm download & extract (`Installable` trait)
  - `service/workflow/`: process lifecycle (Windows no-window: `win_spawn.rs`)
  - `service/cli/`: `dsh`/`pnpm` shims + PATH registration (`mod.rs`/`shim.rs`/`path.rs`/`core.rs`)
  - `service/scheduler/` + `task/`: health check & polling

## Dev Commands

```bash
pnpm install && pnpm dev    # frontend dev
pnpm typecheck              # frontend TS check (must run after frontend changes)
pnpm tauri dev              # full desktop debug
cargo check && cargo test   # Rust check & unit tests (run in src-tauri)
```

## Basics

- No `useCallback` / `useMemo` — project has `react-compiler` built in
- Component functions use `function` declaration; inline events/callbacks use arrow functions

## Function Declaration Specification

- **Named functions must use `function` declaration, not arrow functions**
- **Arrow functions can only be used when passed as callback parameters**

```tsx
// ✅ Correct
function Component() {
  function handleClick() {
    console.log('click');
  }
  return <button onClick={handleClick}>Click</button>;
}

// ✅ Correct: Arrow functions can be used for callbacks
useQuery({
  queryFn: async () => {
    return fetchData();
  },
});
```

## Data Processing Specification

### Use directly in pages

**Use case:** When data doesn't need additional processing after fetching

```tsx
function MyPage() {
  const { data } = useQuery({
    queryKey: ['simple-data'],
    queryFn: () => fetchData(),
  });
}
```

### Create files in services directory

**Use case:** Backend type error handling, parameter processing, composite requests, polling, data caching, etc.

**File naming:** `use-get-{resource}.ts`, `use-post-{resource}.ts`, `use-put-{resource}.ts`, `use-delete-{resource}.ts`

```tsx
// services/use-get-exchange-rates.ts
export function useGetExchangeRates(params) {
  return useQuery({
    queryKey: [getApiExchangeRatesCurrencyPair.name, params],
    queryFn: () => getApiExchangeRatesCurrencyPair(params).then(res => res.data?.data),
  });
}
```

## Conditional Rendering Specification

Use `If`, `Then`, `Else` components by `react-if-lite` package instead of ternary operators and `&&` operators

```tsx
// Basic usage
<If cond={!isLoading} else={<LoadingSpinner />}>
  <Content />
</If>

// Simple condition: use props
<If cond={isBasic} then={<GrayZuanIcon />} else={<ZuanIcon />} />

// Complex condition: use child components
<If cond={hasData}>
  <Then>
    <DataTable data={data} />
  </Then>
  <Else>
    <Empty />
  </Else>
</If>

// Specify render tag
<If cond={condition} as="div">
  {content}
</If>
```

## State Management Specification

### Single module page shared data: defineScope + useScope

```tsx
// 1. Create scope
export const TradingScope = defineScope(() => {
  const [count, setCount] = useState(0);
  // any hooks or functions
  return { count, setCount }; 
});

// 2. Use Provider
<TradingScope.Provider>
  <Page> {/* or ({ count }) => <Page>...</Page> */}
    <TradingTable />
  </Page>
</TradingScope.Provider>;

// 3. Use in child components
const { filters, setFilters } = useScope(TradingScope);
```

### Multiple module pages shared data: defineStore

```tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dsh-tauri-desk/deepseek-harness-desktop](https://github.com/dsh-tauri-desk/deepseek-harness-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
