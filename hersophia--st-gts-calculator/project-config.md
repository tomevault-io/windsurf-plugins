---
trigger: always_on
description: 当用户输入中明确提及 MVU 时, 你应该参考本文件
---

# MVU 角色卡文件夹

MVU 角色卡文件夹提供了一种存储酒馆角色卡内容的文件结构:

- `角色卡/脚本/*/` 中是角色卡的所有脚本项目
- `角色卡/界面/*/` 中是角色卡的所有前端界面项目
- `角色卡/世界书/*/` 中是角色卡的世界书条目, 即角色卡的设定提示词, 编写角色卡其他内容时需要参考它来了解角色世界设定
- `角色卡/schema.ts` 中是用 zod 4 库书写的角色卡 MVU 变量结构定义
  - 提供给脚本、前端界面导入使用
  - 会在 `pnpm build` 或 `pnpm watch` 时生成对应的 json schema 文件 `角色卡/schema.json`, 便于编写变量初始值文件 initvar.yaml `# yaml-language-server: $schema=schema文件路径`
- `角色卡/界面/store.ts` 中是 pinia 预先写好的获取角色卡消息楼层 MVU 变量方式, 提供给所有前端界面导入使用

当玩家要求编写 MVU 角色卡的脚本、前端界面时, 除了参考`初始模板/脚本`或`初始模板/前端界面`外, 你还应该参考`初始模板/角色卡`中的脚本和前端界面模板.

**要区分单独的脚本、前端界面和为 MVU 角色卡增补脚本、前端界面, 如果用户只是想要编写单独的脚本、前端界面, 则不应参考这个文件.**

## MVU 变量结构

MVU 使用 zod 4 库书写变量结构定义, 这对应于`角色卡/schema.ts`, 例如:

```ts
export const Schema = z.object({
  好感度: z.coerce.number().transform(value => _.clamp(value, 0, 100)),
});
```

你应该要求用户提供变量结构文件或者自行编写, 它应该遵循以下要求:

```yaml
rule:
  - libraries: "`z` from zod and `_` from lodash are available by default, so you can use them directly and should prefer to use them; don't import them in the generated code"
  - zod 4: stick to use zod 4.x, never ever use `.passthrough` or `.strict`!
  - idempotent operation: the schema is intended to parse the updates of the world status incrementally, thus, the output of `Schema.parse(input)` must be a valid input of `Schema.parse` itself; that is, you should use z.transform carefully, keeping `Schema.parse(Schema.parse(input))` equal to `Schema.parse(input)`
  - for number schema: prefer `z.coerce.number()` over `z.number()` whenever you expect a number since it will try to convert the input to a number if it's not a number; but don't use other `z.coerce.xxx()` such as `z.coerce.boolean()`, just use `z.boolean()` directly
  - prefer object schema over array schema: "the array index is hard to understand and maintain, so you should use `物品栏: z.record(z.string().describe('物品名'), z.object({ 描述: z.string(), ... }))` instead of `物品栏: z.array(z.object({ 名称: z.string(), 描述: z.string(), ... }))`"
  - for object schema:
      - fixed required keys + the same type: use `z.record(z.enum(['key1', 'key2', ...]), ${value type})`
        fixed optional keys + the same type: use `z.partialRecord(z.enum(['key1', 'key2', ...]), ${value type})`
        dynamic optional keys + the same type: use `z.record(z.string(), ${value type})`
        fixed required keys + different types: 'use `z.object({ key1: ${type1}, key2: ${type2}, ... })`'
        dynamic keys but some keys are required + the same type: 'use `z.intersection(z.object({ requiredKey1: ${type1}, requiredKey2: ${type2}, ... }), z.record(z.string(), ${value type}))`'
      - on clearable object: 'if the object is clearable by JSON patch `{ "op": "remove", "path": "/path/to/object" }`, set `z.object({ ...prefault for every field }).prefault({})` instead of `z.object({ ... }).optional()` for better compatibility with the incremental update'
  - for special format (rare to happen): prefer `z.templateLiteral` over regex or manual parsing
  - for restrictions: when accepting a update that breaks the schema, users are tend to expect the update takes some effect instead of being discarded completely; therefore, you should try your best to use `z.transform` to convert the broken input to a valid input. For example, if Explorer requests a value to be between 0 and 100, prefer `z.number().transform(value => _.clamp(value, 0, 100))` over `z.number().min(0).max(100)`; if an object could only contain 10 keys, when a new key comes, discard the oldest key instead. **but only impose these restrictions when Explorer requests**
  - on default value:
      - prefer `z.prefault` over `z.default`
      - if a `z.object` or the whole Schema is complicated enough, set `.prefault('${suitable default value}')` or `.or(z.literal('待初始化')).prefault('待初始化')` for every field of it
      - don't set `z.prefault` for other situatioins unless Explorer requests it
  - when to describe: use `z.describe` only when there's no field name to explain the usage of the schema such as the key type of `z.record`; in contrast, you should never use `z.describe` if the field name has already explained the usage well
  - determine the order of keys: 'if Explorer requests you to do something with the insertion time of keys, prefer to use `_(data).entries()` which almost always lists keys in insertion order, e.g. you can remove old keys with a simple `_(data).entries().takeRight(10)`; when keys are already additionally sorted inside `z.transform`, you should use `$time: z.coerce.number().prefault(() => Date.now())` to automatically assign a timestamp'
  - don't repeat yourself: merge the same variable schemas whenever possible, but don't define extra variables to do so - you can only define schema inside `export const Schema = z.object({ ... })`
  - type of functions:
      - '`registerMvuSchema = (schema: z.ZodObject | (() => z.ZodObject)) => void`, the function input `() => z.ZodObject` could be used when the schema is not ready at the time of registering, or the schema depends on runtime data/function'
      - '`z.transform(value => value)`'
      - '`z.prefault/z.catch(value | () => value)`'
  - REPEAT: DON'T mark any variable as optional or impose any restriction to the schema unless Explorer requests them
```

如果用户提供了 `export const Schema`, 你应该区分用户提供的是直接的 `schema.ts` 还是变量结构脚本. 具体地,


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HerSophia/ST-GTS-Calculator](https://github.com/HerSophia/ST-GTS-Calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
