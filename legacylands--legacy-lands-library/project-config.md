---
trigger: always_on
description: > 本规范适用于你参与的全部开发任务，条款为强制性，除非用户显式豁免，任何条目都不得忽视或删减。
---

# ✅ 开发与协作规范（学习与编程行为指南）

> 本规范适用于你参与的全部开发任务，条款为强制性，除非用户显式豁免，任何条目都不得忽视或删减。
>
> - 每次输出前必须深度理解项目背景、用户意图和技术栈特征；
> - 当回答依赖外部知识或信息不确定时，先查询至少一份权威资料（官方文档、标准或源码）再作结论；
> - 引用外部资料须在回复中注明来源；可使用链接或版本号，保证可追溯；
> - 若用户需求含糊，先用一句话复述已知信息，并列出关键澄清问题，待用户确认后再继续；
> - 同一次回复中的术语、变量名、逻辑描述保持一致；发现冲突必须立即修正；
> - 仅回答与问题直接相关内容，避免冗余、无关扩展或教程式铺陈；
> - 面对复杂需求，先拆分为若干可管理子任务，在输出中按子任务顺序呈现，便于用户跟进；
>
> 所有技术输出必须建立在准确、思考过的基础之上，拒绝机械生成与无脑填充。
> 如果你已经了解所有的规范，你需要在用户第一次进行对话时先说明 "我已充分了解开发与写作规范。"，随后再执行用户需求。

## 1. 当你尝试编写一个新功能、添加新类或书写 Javadoc 时

1. 你需要先查看其他类的实现，尤其是相同模块中的类；
2. 主动学习、模仿现有代码的结构与风格，包括：
    - 缩进与换行格式；
    - 命名习惯；
    - Javadoc 注释风格。
3. 多使用 `serena` MCP 完成可能的操作。

## 2. 你必须制定清晰的 ToDo 任务清单，并在开发完成后遵循以下规则：

1. 使用 Gradle 的 `shadowJar` Task 尝试打包；
2. 若出现编译错误，必须修复后重新编译，直至编译通过；
3. 编写测试前，先熟悉 `foundation` 模块以及 `net.legacy.library.annotation.test` 包中已提供的测试示例；
4. 编写基于功能的集成测试，测试内容不得流于无意义断言，需覆盖核心业务逻辑与边界条件；
5. 测试通过后，在当前模块的 `Launcher` 中注册新测试入口，参考 `net.legacy.library.annotation.AnnotationLauncher` 的实现方式；
6. 若需要为当前模块引入其他模块作为依赖：
    - 先检查是否会造成循环依赖；
    - 若无循环依赖，则使用 `compileOnly` 方式声明；
7. 编译全部通过后，将完整测试流程交由用户执行，由用户反馈日志进行进一步调整，你不能自己执行任何的测试，包括 `javap` 等指令；
8. 除非用户有明确说明，否则你不应撰写任何示例（example）或额外文档。

## 3. 命名规范（类、字段、变量、方法）

1. 命名应清晰、准确、语义明确：
    - ✅ 正确示例：`Exception exception`，`Throwable throwable`，`IOException exception`
    - ❌ 错误示例（过于简单化）：`Exception e`，`Throwable t`
    - ❌ 错误示例（过于复杂化）：`Exception exceptionWhenFailedToInitializeXxx`
    - ❌ 错误示例（无意义）：`Cache whatIsThis = ...`
2. 出现命名重复时，应提升语义层级，而非简单加数字：
    - ✅ 正确示例：`Cache cache; Cache redisCache`
    - ❌ 错误示例：`Cache cache; Cache cache2`

## 4. 数据处理优先使用 `Stream API`（尤其是过滤、映射、聚合等操作）

- 避免使用传统 `for` 循环完成可以用 `stream` 完成的链式操作；
- 确保代码简洁、函数式、可读性强。

## 5. Lambda 表达式需保持简洁，避免冗余语法

1. ✅ 正确用法（单行时省略 `{}` 与 `return`，可使用方法引用）：
    - `() -> "something"`
    - `list.forEach(CustomExecutor::shutdown)`
2. ❌ 错误用法（冗余、过度包装）：
    - `(x) -> { doSomething(x); }` → 简化为 `x -> doSomething(x)`
    - `() -> { return "something"; }` → 简化为 `() -> "something"`
    - `list.forEach(x -> { CustomExecutor.shutdown(x); });` → 应使用方法引用

## 6. Javadoc 编写规范

1. 你必须优先参考 `net.legacy.library.annotation.service` 包下的类作为注释风格基准；
2. 若该包中无类似内容，再自行查阅其他类学习注释方式，但必须保持风格一致；
3. 常规结构应包括：
    - 简洁说明性首句（以动词开头），句末加英文句号；
    - 标签顺序统一：`@param` → `@return` → `@throws`；
    - 不遗漏边界条件、默认行为或异常说明；
4. 禁止使用不规范注释格式，特别是：
    - ❌ `@param input the input`
    - ✅ `@param input the user-defined script to evaluate`
5. 如果原始 Javadoc 中存在不合理的格式，应记录问题并征求负责人同意后统一修正；
6. 若方法涉及复杂逻辑，必须标明：
    - 是否线程安全；
    - 对 null 的处理策略；
    - 默认行为或返回值；
    - 抛出异常的触发条件与意义；
7. 必须模仿已有项目注释中的语气、缩进、空行规则，不得混用不同风格；
8. 如果使用接口、抽象类、继承等，重写的方法若没有需要注重的内容，则可以对形参、返回值、异常、描述等等使用 `{@inheritDoc}`；
9. 禁止写没必要的 `Usage example`；
10. 不需要为非 `public` 字段或方法写 Javadoc，除非真的有需要；
11. 其他标签使用规范：
    - `@see`：列出相关类、接口或方法，按出现顺序书写；
    - `@since`：首次引入版本；
    - `@deprecated`：标记过时元素，并使用 `@see` 指向替代方案；
    - `@implSpec`：说明实现细节，供继承或实现者阅读；
    - `@implNote`：记录实现者备注（性能、限制等），不影响规范语义；
    - `@apiNote`：公开 API 额外说明（示例、用法注意事项等）；
12. 内联标签使用规范：
    - `{@link ClassName#memberName}`：在描述中插入可点击引用，避免冗长全名。例如：  
      `Parses the given {@link java.nio.file.Path} instance.`
    - `{@code ...}`：在描述或列表中嵌入等宽代码片段。例如：  
      `Returns {@code null} if the input is empty.`
    - `{@literal ...}`：插入按字面量输出的字符串，避免被解析为 HTML/标签：  
      `Converts the string {@literal "<none>"} to an empty value.`
    - `{@value #CONSTANT}`：引用常量值，保持文档与代码同步。
13. 标题、列表、段落等 HTML 标签仅在确实提升可读性时使用，禁止为追求样式而滥用。
14. 使用任何标签或 HTML 元素时应遵循 "必要且充分" 原则；

## 7. Git 提交规范（Commit Message）

1. 提交信息必须简洁明确，并符合项目现有格式风格。你需要：
    - 查看近期的 Git 历史记录（`git log --oneline` 或 `git log -n 10`）；
    - 学习当前项目所使用的 commit 语言风格、结构模板等；
    - 避免使用口语、不相关内容或个人语气；
    - 若项目采用语义化提交规范（Conventional Commits），你必须严格遵守其格式。
2. 通用规范（若无特殊格式要求）：
    - ✅ 正确示例：
        - `Fix: resolve NPE in TaskHandler`
        - `Refactor: extract method from UserSessionManager`
        - `Add: support for async task execution`
        - `Docs: update Javadoc for ConfigLoader`
    - ❌ 错误示例：
        - `修复一下问题`
        - `update`
        - `提交代码`
3. 推荐格式（若尚无约定）：
    ```
    <类型>: <简要说明（英文首字母大写）>
   
    <可选说明：描述变更原因、影响范围、解决的问题等>
    ```

   常见类型包括：
    - `Add:` 新增功能、类、方法
    - `Fix:` 修复 bug
    - `Refactor:` 重构（不涉及功能行为改变）
    - `Docs:` 修改文档或注释
    - `Test:` 添加或修复测试
    - `Chore:` 杂项、配置项更新等

4. 合并（merge）请求前，务必 squash commit 或整理历史，保持提交记录清晰有序。

## 8. 自主学习与风格一致性要求

1. 你必须具备 "自我学习、自我适配" 的意识，凡遇风格不确定、规则模糊的场景，须优先：
    - 主动查阅现有代码；
    - 模仿当前模块的实现方式；
    - 避免引入破坏风格一致性的代码。
2. 遇到新模块、空白模块时：
    - 可以参考 `net.legacy.library.*` 包中的类为基本风格模板；
    - 所有新的命名风格、设计结构，必须与项目现有命名方式协调一致；
    - 若存在不确定性，且确认已经进行学习后仍然不清楚，可暂时标记 `// TODO: 确认命名规范` 后告诉开发组，待开发组确认后再定稿。
3. 在大型重构或多人协作中，应保持沟通、拉取最新代码，确保提交不会引起大面积冲突或风格割裂。
4. 在开始编码前，必须熟悉并优先使用以下已有模块功能，避免重复实现：
    - 注解相关：查看 `net.legacy.library.annotation` 及其子包（尤其是 `annotation.service`）中的现有工具和处理器；
    - 线程调度与并发工具：查看 `net.legacy.library.commons` 包内提供的线程池、调度器等实现；
    - 缓存与分布式锁：查看 `net.legacy.library.cache` 包下的缓存包装、锁框架等；
    - 其他通用功能（如 I/O、JSON、配置解析），先在 `net.legacy.library.*` 层级中检索。
5. 若现有模块已满足需求，禁止自写替代实现；如确认存在缺口，需在 Pull Request 描述和 commit 信息中同时说明：
    - 已检索过的相关类或方法；
    - 为什么现有实现不足；
    - 新实现的范围与改进点；
    - 例：`Add: extend cache layer to support LFU (#1234)`。
6. 对任何新功能，先阅读相关模块的 README / Javadoc / 示例，确保理解其使用方式与边界，再决定是否扩展。
7. 只有当现有实现无法满足需求或存在严重缺陷时，才可自行重构或实现，并必须在 commit 信息与 PR 描述里写明评估过程与理由，避免后续重复开发。

## 9. 注释与回答语言规范

### 9.1 注释语言统一为英文

1. 项目代码中的注释必须使用英文，除非明确说明或学习中确定项目注释大多为中文，包括但不限于：
    - 类注释；

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LegacyLands/legacy-lands-library](https://github.com/LegacyLands/legacy-lands-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
