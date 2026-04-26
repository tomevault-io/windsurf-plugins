---
trigger: always_on
description: 一个通用的 LLM Agent 桌面客户端
---


## 技术栈

### 项目组织

这个项目使用 Tauri v2 构建，使用额外的 Python 子项目作为 Sidecar。
使用 Nx 来进行多语言间的命令管理。

### 前端部分

前端框架：React 19
样式：TailwindCSS
状态管理：zustand
组件库：Shadcn-ui + Vercel AI Elements
TypeScript Schema 生成和 API 请求函数生成：orval
数据校验：zod

### Python Sidecar

后端框架：FastAPI
数据库：SQLite
ORM：SQLAlchemy + Alembic
数据校验：Pydantic

## 关键技术点实现

### 数据持久化

仅前端使用的数据直接使用 zustand 的 persist 存储在 localStorage 中。
对于前后端共享数据，如工作区设置、Agent 配置、用户会话数据等，通过后端存储在 SQLite 数据库中。

## 开发指南

### 上下文信息要求

- 在编码前至少分析 3 个现有实现或模式，识别可复用的接口与约束。
- 绘制依赖与集成点，确认输入输出协议、配置与环境需求。
- 弄清现有测试框架、命名约定和格式化规则，确保输出与代码库保持一致。

### 代码风格

#### 前后端共用代码风格

- 字符串统一使用双引号

#### 前端代码风格

- **类型**: 使用 TypeScript 编写，禁止使用 any
- **样式规范**: 使用 TailwindCSS 类名，禁止写 css 文件
- **页面路由**: 在 `frontend/src/pages/` 新增组件即自动成为路由
- **组件编写**: 统一使用 `function Component(props: Props) {}` 格式编写，不要使用箭头函数
  - **图标组件使用**: 统一使用 lucide-react 库提供的图标组件，且使用时需要带 `Icon` 后缀。（例如，使用 `Plus` 图标应该导入 `PlusIcon`）
  - **dialog 组件使用**: 在创建通用 dialog 组件时，需要基于 shadcn 提供的 dialog 组件创建，同时必须使用 DialogTrigger 来控制 dialog 状态，不要在 dialog 组件外额外使用 useState 管理 dialog 状态
- **错误处理**: 在出现错误时统一使用 toast 组件展示

#### 后端代码风格

- **语法**:
  - 对于一行拆分为多行的语句，应该遵循 PEP 8 规范，使用 `()` 进行包裹，而不是使用 `\` 进行拆分。
- **类型定义**:
  - 不应该使用 `Option[T]`, `Union[A, B]`, `List[T]`, `Dict[T]` 等过时的类型标注写法
  - 应该使用 `T | None`, `A | B`, `list[T]`, `dict[T]` 等更现代的写法
- **导入语句**: 遵循 Python 最佳实践
  - 导入顺序：标准库 -> 第三方库 -> 本地应用/库导入
  - 对于每个包的导入应该独占一行（使用 `\` 符号换行除外）
  - 应该避免使用通配符导入（from module import *）
  - 如果在一个 Python 模块中使用了一个包的多个导出，应使用“from ... import ...”；如果仅使用一个包的一个导出，则使用 "import ..."
- **错误处理**:
  - 禁止使用 bool 值或 None 来表示出错
  - 在需要时定义自定义错误类型来明确错误原因
  - 在 API 路由处将错误转换为错误代码，供前端转换为用户易懂的错误原因
  - 在直接忽略错误时，必须给出解释说明
  - 错误日志应该仅出现在 except 块中，即实际处理错误的块中，raise 语句处禁止打错误日志

### 最佳实践

#### 后端

- **路由函数返回值**: 应一律在在路由装饰器中定义 response_model，并在 returns 时直接返回 Pydantic Model。
- **状态码**: 在需要使用 HTTP 状态码时不应直接写魔法数字，应该使用 FastAPI.status 中的状态码常量（比如 `status.HTTP_204_NO_CONTENT`）。
- **错误处理**: 在 Service 层应该直接自定义错误类型，不应继承框架提供的错误类型，在创建 app 时使用统一的 exception_handler 来将 service 层错误类型转换为框架层的错误类型并返回给前端。
- **Service 注入**: 在路由中如果需要调用 service，需要使用 FastAPI 的 Depends 装饰器配合 `Annotated` 注入 service，示例代码：
```python
agents_router = APIRouter()

def get_agent_service():
    with AgentService() as service:
        yield service

AgentServiceDep = Annotated[AgentService, Depends(get_agent_service)]

@agents_router.get("/{agent_id}", response_model=AgentRead)
def get_agent(agent_id: int, service: AgentServiceDep):
    agent = service.get_agent_by_id(agent_id)
    return AgentRead.model_validate(agent)
```

- 在 service 中：
  - 对于 create 的方法，应该在创建后直接再次调用 get_xxx_by_id 方法来获取并返回创建的对象
  - 对于 update 的方法，应该在更新后先调用 `expunge` 方法来清除缓存，随后再调用 get_xxx_by_id 方法来获取并返回更新后的对象

#### 前端

- 在调用 API 方法时，对于从后端拉取状态的方法，使用 `@tanstack/react-query` 的 `useQuery` 来管理请求状态；对于需要修改后端状态的方法，使用 `@tanstack/react-query` 的 `useMutation` 来管理请求状态。
- 在编写需要请求后端获取数据来渲染的组件时，需要将具体 fetch 后端 API 并渲染数据的部分封装成子组件，在子组件中使用 `@tanstack/react-query` 的 suspense API 来请求后端，并在父组件中使用 `<Suspense>`、`<ErrorBoundary>` 和 `@tanstack/react-query` 提供的 `QueryErrorResetBoundary` 组件来包裹子组件并处理加载、错误和重试等状态。
- 在创建表单时，应使用 Composable Form 范式，即：
  - 拒绝巨型表单：禁止在单一组件内通过大量的 `if (isEditMode)` 这样的条件判断来控制字段渲染。
  - 组合优于配置：使用“积木式”组合，由容器组件（Container）决定字段的渲染顺序和存在性。
  - 原子化表单组件：将特定业务字段的 UI (Input, Select 等)、逻辑 (Controller) 和校验规则 (Rules) 封装为独立组件。
  - 上下文驱动：使用 FormProvider (在表单 Container 中) 和 useFormContext (在 Field Component 中) 避免 Props Drilling。
- 在编写资源选择 dialog 组件时，应该基于项目现成的 SelectDialog 组件构建，同时对于需要异步加载的数据，应该单独将选项列表封装在单独的 list 组件中，并在该组件中使用 suspense API 加载数据，同时在 dialog 层处理 suspense 状态。

### 注意事项

#### UI 风格细节

- **按钮英文文本大小写**：使用“句子大小写”，如 "Add to cart", "Create new account"
- **设置界面设置项标题英文文本的大小写**：使用“句子大小写”
- **对话框 (dialog) 组件标题英文文本的大小写**：使用“句子大小写”
- **Label 组件英文文本的大小写**：使用“句子大小写”

#### 后端部分

- 禁止直接修改 `alembic/versions` 目录下的数据库迁移文件

#### 前端部分

注：下面提到的依赖默认已经存在，不需要再去读文件验证是否存在。

- 写组件时优先查找 shadcn 中是否有现成组件，有的话直接导入使用。没有的话再自行封装组件。
- 编写样式时要合并 Tailwind CSS 类时使用 `cn()` 工具函数，已位于 `src/lib/utils.ts` 中。
- 要使用图标时从 `lucide-react` 库中，统一用 `import { IconName } from "lucide-react"` 进行导入。
- 使用 `zustand` 库进行状态管理，全局 store 放在 `frontend/src/stores/xxxStore.ts` 中。
- 在进行 API 请求时，需要在 `frontend/src/api` 目录下创建新的 API 文件，API 方法统一使用 `function` 关键字定义。
- 在需要创建表单时，使用 `react-hook-form` 库来辅助创建，并使用 shadcn-ui 提供的 Field 组件来辅助构建表单组件。
- 在表单校验的 pattern 中，如非必要禁止直接编写正则表达式，应该一律使用 `validator.js` 库提供的方法进行校验
- **严禁修改 `components` 目录下的文件**

需要忽略的情况：
- 如果遇到 CSS 类顺序错误请直接忽略。用户会在开发完成后一并解决。
- 如果遇到 `components/ui` 中的组件报错，请忽略。

#### 前端 Biome 代码规范

Avoid `accessKey` attr and distracting els
No `aria-hidden="true"` on focusable els
No ARIA roles, states, props on unsupported els
Use `scope` prop only on `<th>` els
No non-interactive ARIA roles on interactive els
Label els need text and associated input
No event handlers on non-interactive els
No interactive ARIA roles on non-interactive els
No `tabIndex` on non-interactive els
No positive integers on `tabIndex` prop
No `image`, `picture`, or `photo` in img alt props
No explicit role matching implicit role
Valid role attrs on static, visible els w/ click handlers
Use `title` el for `svg` els
Provide meaningful alt text for all els requiring it
Anchors need accessible content
Assign `tabIndex` to non-interactive els w/ `aria-activedescendant`
Include all required ARIA attrs for els w/ ARIA roles
Use valid ARIA props for the el's role
Use `type` attr on `button` els

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dais-Project/Dais](https://github.com/Dais-Project/Dais) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
