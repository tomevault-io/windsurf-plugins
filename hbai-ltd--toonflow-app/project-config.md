---
trigger: always_on
description: 1. 优先使用简洁直接的实现方案：减少嵌套层级、删除冗余分支、避免不必要的抽象，保证代码可读性优先。
---

本规范适用于整个仓库。

# 开发规范说明

## 代码风格要求

1. 优先使用简洁直接的实现方案：减少嵌套层级、删除冗余分支、避免不必要的抽象，保证代码可读性优先。
2. 函数保持小而聚焦，仅当复用价值或可读性有明确提升时才做逻辑抽取。
3. 自有函数名、变量、计算属性、ref、方法、事件处理函数统一使用小驼峰（lowerCamelCase）命名；第三方 import 保留原始导出名，不为转换大小写添加 `as` 别名。
4. 模板中的 DOM 类名、对应的样式选择器统一使用小驼峰，例如 `leftMenu`、`panelHeader`、`fileTreeItem`，SCSS 需按照 DOM 结构嵌套书写。
5. 不使用全大写常量，优先使用语义化小驼峰命名，例如 `editorConfig`、`requestTimeout`、`panelWidth`。
6. 新增逻辑前优先复用已有的工具函数和 Store 方法，避免新增不必要的工具层。
7. 所有新建文件、文件夹名称必须统一使用小驼峰**，无任何例外，例如 `userStore.ts`、`fileTree.ts`、`apiHelper.ts`、`editorPanel/`、`contextMenu/`。
8. **禁止使用短横线命名、蛇形命名、帕斯卡命名或全大写格式作为文件/文件夹名**，项目中已存在的例外情况不作为新开发的参考依据。
9. **组件文件名同样严格遵循小驼峰规则**，例如 `tabHeader.ts`、`splitPane.ts`、`loginForm.ts`，禁止使用 `TabHeader.ts` 或 `tab-header.ts` 这类格式。
10. **自有组件（项目内自己编写的组件）的文件名、本地绑定名和模板标签必须使用小驼峰**，例如 `showBox.vue`、`import showBox from "./showBox.vue"`、`<showBox />`，不得使用 `<show-box />`。
11. **第三方 UI/组件库的模板标签允许使用短横线分隔（kebab-case）或小驼峰，优先统一使用短横线分隔**，例如 `<el-button />`、`<vue-flow />`、`<icon-map />`；此规则不放宽自有文件、变量或 DOM 类名的小驼峰要求。
12. **所有组件模板标签及自有组件本地绑定名绝对禁止大驼峰（PascalCase）**，例如禁止 `<ShowBox />`、`<ElButton />`、`<VueFlow />`。第三方组件直接按库原始导出名导入，例如 `import { ElButton } from "element-plus"`、`import { VueFlow } from "@vue-flow/core"`，模板分别使用 `<el-button />`、`<vue-flow />`；脚本及模板表达式直接使用原始导出名，不添加仅用于转小驼峰的 `as` 别名。类型名、库导出名及工具自动生成的声明不属于模板标签，不手工改写自动生成文件。
13. **所有 `.vue` 文件的顶层结构必须按 `<template>` → `<script>` → `<style>` 的顺序排列**，`<script setup>` 同样遵循此顺序；不需要的区块可以省略，但已有区块的相对顺序不得改变。
14. **所有组件的属性名必须统一使用小驼峰，包括自有组件、第三方组件的 props 声明、静态属性和动态绑定**，例如 `showArrow`、`:nodeTypes`、`:snapToGrid`，禁止写成 `show-arrow`、`:node-types`、`:snap-to-grid`。具名 `v-model` 的参数同样使用小驼峰，例如 `v-model:snapEnabled`。组件标签允许短横线的规则不适用于属性名。
15. **仅 Vue 语法、HTML 标准或第三方接口强制要求的名称保留原始写法**，例如 `v-if`、`v-for`、`v-model`、`v-bind`、`v-on`、`aria-label`、`data-*`；不得将这些名称改为小驼峰。第三方文档中的短横线示例不构成例外，支持小驼峰的组件属性仍必须使用小驼峰。

## 前端工作区文件操作

- `apps/web/src/lib/workspaceFiles.ts` 默认导出 `useWorkspaceFiles`。组件与前端工具统一复用此入口，不重复封装 Axios 或直接拼接 `/api/workspaces/files/*` 请求。
- 方法中的 `path`、`target` 均为工作区内的相对路径，例如 `画布1.json`、`assets/image.png`；目录参数使用绝对路径。
- 不传目录时，使用 Pinia 中当前项目的工作目录，每次操作重新读取；在 Pinia 初始化后的组件 `setup` 中创建实例。未选择工作目录时操作报错。

```ts
import useWorkspaceFiles from "@/lib/workspaceFiles";

const files = useWorkspaceFiles();
const { directory, entries } = await files.list();
const content = await files.readText("说明.txt");
await files.write("说明.txt", content);
```

### 目录绑定

- `useWorkspaceFiles(directory)`：传入目录字符串，固定该实例的目标目录；普通 TypeScript 工具函数可直接使用，不依赖当前 Pinia 实例。
- `useWorkspaceFiles(directoryRef)` 或 `useWorkspaceFiles(() => props.directory)`：传入 ref 或 getter，每次操作读取最新值；显式目录为空时直接报错，不回退到当前项目。
- 新建项目尚未更新 Pinia 时，显式传入用户选择的目录；`list()` 返回服务端规范化后的 `directory`，后续创建与失败回滚使用同一规范目录。
- 防抖、保存队列、跨 `await` 的多步操作必须在操作开始时取得目录字符串快照，后续步骤复用固定目录实例，避免切换项目后读写到另一个目录。自动保存与防抖仍由所属页面管理，文件封装不自动监听或保存数据。

```ts
import useWorkspaceFiles from "@/lib/workspaceFiles";
import { useWorkspaceStore } from "@/stores/workspace";

const workspaceStore = useWorkspaceStore();

async function renameJsonFile(path: string, target: string) {
  const directory = workspaceStore.project?.directory;
  if (!directory) throw new Error("请先选择工作目录");
  const files = useWorkspaceFiles(directory);
  await files.rename(path, target);
  return files.readJson(target);
}
```

### 方法与返回值

所有文件操作均返回 Promise；写入、改名、删除、建目录成功时无返回内容。

| 方法 | 用法与返回值 |
| --- | --- |
| `list(path = "")` | 列出一层目录，返回 `{ directory, entries }`；每项包含 `name`、相对 `path`、`type`（`file` 或 `directory`）。 |
| `read(path)` | 读取二进制，返回 `ArrayBuffer`。 |
| `readText(path, maxBytes?)` | 读取文本，返回字符串；传入正整数 `maxBytes` 时通过 HTTP Range 只读取文件头指定字节数。 |
| `readJson<T = unknown>(path)` | 读取并解析 JSON，返回 `T`；泛型仅提供类型提示，不校验文件结构。 |
| `write(path, content, exclusive = false)` | 写入字符串、`Blob` 或 `ArrayBuffer`；默认创建或覆盖整个文件，第三个参数传 `true` 时只允许新建。 |
| `writeJson(path, data, exclusive = false)` | 将数据格式化为 JSON 后写入；第三个参数传 `true` 时只允许新建。 |
| `rename(path, target)` | 在工作区内改名或移动文件、目录；目标已存在时不覆盖。 |
| `remove(path, recursive = false)` | 删除文件或空目录；显式传 `true` 才递归删除目录内容。 |
| `mkdir(path)` | 创建目录；父目录须存在，不自动递归创建。 |

- 文件的标记字段和业务结构由调用方负责，例如画布的 `toonflowCanvas`、对话的 `toonflowAgent`；不能因调用了 `readJson<T>` 就假定结构有效。
- 所有请求错误原样抛给调用方处理，JSON 解析失败抛出 `SyntaxError`；不要吞掉写入失败或无条件重试。新增文件的自动编号只处理服务端明确返回的同名冲突。
- 此封装只负责工作区文件；全局设置继续使用设置接口，项目列表继续由 Pinia 持久化，移除列表项不等于删除工作区文件。

## Server 开发规范

以下规则适用于 `apps/server`，与上面的通用代码规范同时遵守。

### 技术栈与职责

- 使用 Bun、TypeScript、ES Modules 和 Express，沿用现有依赖与工具，不另建服务框架。
- `src/index.ts` 是独立 server 的启动入口，单进程监听端口。
- `src/app.ts` 的 `createApp({ webRoot, dataDirectory?, ... })` 负责创建应用、装配中间件、静态资源、路由和统一错误处理，返回应用；传入的数据目录须在动态加载路由前设置。不要在这里启动监听或创建 worker。
- 桌面端通过 `@toonflow/server/app` 复用应用，不导入独立 server 的启动入口，不额外启动 cluster。

### 目录结构

```text
apps/server/
  package.json
  tsconfig.json
  src/
    index.ts                 # 独立服务启动
    app.ts                   # Express 应用装配
    core.ts                  # 根据文件目录生成路由
    router.ts                # 自动生成的路由注册文件
    utils.ts                 # 通用工具统一出口，默认导出对象
    utils/
      conf/index.ts          # conf 实例与配置
      mcp/                   # MCP 控制、工具和资源
    lib/
      middleware.ts          # 参数校验等 HTTP 中间件
      responseFormat.ts      # 统一响应格式
    routes/
      hello.ts               # 单个接口
      settings/              # 按业务分类
        get.ts               # 读取设置接口
        save.ts              # 保存设置接口
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HBAI-Ltd/Toonflow-app](https://github.com/HBAI-Ltd/Toonflow-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
