---
trigger: always_on
description: **Scope:**All files matching `**/*.{ts,tsx}`
---

## Code Style Guidelines

**Scope:**All files matching `**/*.{ts,tsx}`

**Purpose:** Code Style & Structure specifics

### Code Style

- Write concise, functional code with proper types

  ```ts
  // Good
  function mergeConfigs<T>(base: T, override: Partial<T>): T {
    return { ...base, ...override }
  }

  // Avoid
  class ConfigMerger {
    merge(base: any, override: any) {
      return Object.assign({}, base, override)
    }
  }
  ```

- Use Bun native modules when available

  ```ts
  // Good
  import { file } from 'bun'

  // Avoid
  import { readFile } from 'node:fs/promises'
  const config = await file('config.json').json()
  const config = JSON.parse(await readFile('config.json', 'utf-8'))
  ```

- Use descriptive variable names with proper prefixes

  ```ts
  // Good
  const isConfigValid = validateConfig(config)
  const hasCustomOptions = Boolean(options.custom)
  const shouldUseDefaults = !configExists || isConfigEmpty

  // Avoid
  const valid = check(cfg)
  const custom = !!options.custom
  const defaults = !exists || empty
  ```

- Write proper JSDoc comments for public APIs

  ```ts
  /**

   * Loads configuration from a file or remote endpoint
   * @param options - Configuration options
   * @param options.name - Name of the config file
   * @param options.cwd - Working directory (default: process.cwd())
   * @returns Resolved configuration object
   * @throws {ConfigError} When config loading fails
   * @example
   * ```ts
   * const config = await loadConfig({
   * name: 'myapp',
   * defaultConfig: { port: 3000 }
   * })
   * ```

   */
  async function loadConfig<T>(options: Config<T>): Promise<T>
  ```

- Use proper module organization

  ```ts
  export { ConfigError } from './errors'
  // config.ts
  export { loadConfig } from './loader'
  export type { Config, ConfigOptions } from './types'
  ```

- Follow consistent error handling patterns

  ```ts
  // Good
  const result = await loadConfig(options).catch((error) => {
    console.error('Config loading failed:', error)
    return options.defaultConfig
  })

  // Avoid
  try {
    const result = await loadConfig(options)
  }
  catch (e) {
    console.log('Error:', e)
  }
  ```

- Use proper type assertions

  ```ts
  // Good
  const config = result as Config
  if (!isValidConfig(config))
    throw new Error('Invalid config')

  // Avoid
  const config = result as any
  ```

## Documentation Guidelines

**Scope:**All files matching `**/*.{ts,tsx,md}`

**Purpose:** Documentation specific rules

### API Documentation

- Document all public APIs thoroughly
- Include TypeScript type information
- Provide clear function signatures
- Document config options and defaults
- Include return type information
- Document async behavior

### Configuration Examples

- Provide basic usage examples
- Include complex configuration examples
- Document all supported config formats
- Show browser usage examples
- Include TypeScript configuration examples
- Document config merging behavior

### Type Documentation

- Document generic type parameters
- Explain type constraints
- Document interface properties
- Include type union explanations
- Document type generation features
- Provide type utility examples

### Error Documentation

- Document common error scenarios
- Include error handling examples
- Document error recovery options
- Explain validation errors
- Document browser-specific errors
- Include troubleshooting guides

### Code Examples

- Include runnable code examples
- Provide TypeScript examples
- Show error handling patterns
- Include browser environment examples
- Document testing approaches
- Include CLI usage examples

### Best Practices

- Keep documentation up to date
- Use consistent formatting
- Include inline code comments
- Document breaking changes
- Maintain a changelog
- Include version information

### File Structure

- Maintain clear docs organization
- Use proper markdown formatting
- Include table of contents
- Organize by topic
- Keep related docs together
- Use proper headings

### Documentation Standards

- Use clear and concise language
- Include proper code blocks
- Document all parameters
- Provide return value descriptions
- Include usage notes
- Document dependencies
- Keep examples current

## Error Handling Guidelines

**Scope:**All files matching `**/*.{ts,tsx}`

**Purpose:** Error Handling and Validation specifics

### Error Handling

- Use early returns and guard clauses for validation

  ```ts
  function loadConfig<T>(options: Config<T>) {
    if (!options.name)
      throw new Error('Config name is required')

    if (!isObject(options.defaultConfig))
      throw new Error('Default config must be an object')

    // Continue with valid input
  }
  ```

- Implement proper error types

  ```ts
  class ConfigError extends Error {
    constructor(
      message: string,
      public readonly code: string,
      public readonly details?: unknown
    ) {
      super(message)
      this.name = 'ConfigError'
    }
  }
  ```

- Use descriptive error messages

  ```ts
  throw new ConfigError(
    `Failed to load config file: ${filePath}`,
    'CONFIG*LOAD*ERROR',
    { cause: error }
  )
  ```

- Handle async errors properly

  ```ts
  async function loadConfigFile(path: string) {
    try {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stacksjs/buddy-bot](https://github.com/stacksjs/buddy-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
