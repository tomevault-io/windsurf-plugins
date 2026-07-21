---
trigger: always_on
description: 1. **强制使用中文**：所有回答、思考过程、输出内容必须使用中文
---

# 交互语言要求

1. **强制使用中文**：所有回答、思考过程、输出内容必须使用中文
2. **代码除外**：代码本身和特殊专有名词（如类名、方法名、API 名称）使用英文
3. **展示思考过程**：在回答问题时，需要展示分析和推理过程

## 补充强化（必须遵守）

1. **不得直接粘贴英文原始输出**：包括但不限于后台 Agent、外部文档、命令行输出等；如包含英文内容，必须先用中文进行归纳/翻译后再输出。
2. **面向用户的最终输出必须中文**：允许在代码块、类名/方法名/API 名称、URL、HTTP Header 等专有名词处使用英文。
3. **“展示思考过程”的边界**：以“结论 → 证据 → 推理 → 验证步骤 → 规避方案”的结构化方式说明；避免输出无关的内部草稿或逐字推演。

---

# AGENTS.md - AI 股票交易系统开发指南

## 项目结构

本项目采用前后端分离架构，包含以下两个主要部分：

```
stock-trading/
├── backend/                        # Java Spring Boot 单体应用
│   ├── src/main/java/com/stock/
│   │   ├── config/                # 全局配置
│   │   ├── dataCollector/        # 数据采集
│   │   ├── modelService/         # AI 模型（LSTM、情感分析）
│   │   ├── strategyAnalysis/    # 策略分析
│   │   ├── tradingExecutor/     # 交易执行（含 job 调度）
│   │   ├── event/、handler/、logging/、service/
│   │   └── ...
│   └── pom.xml
│
├── frontend-v2/                    # React + Vite 前端
│   ├── src/pages/、components/、layouts/、store/、locales/
│   ├── package.json、vite.config.ts
│   └── ...
│
├── models/sentiment/               # 情感模型（TorchScript，Git LFS）
├── docs/                           # 文档（00~04 模块需求与设计）
├── .tmp/                           # 临时文件（不提交）
├── docker-compose.yml
└── pom.xml
```

**临时文件约定**：项目根目录下的 `.tmp/` 用于存放所有临时文件和中间脚本。Agent 生成的一次性脚本、临时输出、中间结果等均应放在此目录，且该目录已加入 `.gitignore`，不纳入版本控制。

## 构建与运行命令

### Backend (Java/Maven)

工作目录: `backend/`

```bash
# 启动应用 (Spring Boot)
mvn spring-boot:run

# 构建打包 (跳过测试)
mvn clean package -DskipTests

# 编译代码
mvn compile
```

### Frontend (React/Vite)

工作目录: `D:\ai-stock-trading\frontend`

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 代码检查 (Lint)
npm run lint
```

## 测试说明

本项目**不维护自动化测试**（无单元测试、集成测试）。原因与说明见 [README - 关于测试](./README.md#关于测试)。构建时使用 `mvn package -DskipTests` 跳过测试。

## 代码规范

### Java (Backend)

- **版本**: Java 17, Spring Boot 3.2.x
- **风格**:
  - 类名 `PascalCase`, 方法/变量 `camelCase`, 常量 `UPPER_SNAKE_CASE`
  - 使用 Lombok (`@Data`, `@Slf4j`, `@RequiredArgsConstructor`) 简化代码
  - Controller 返回统一泛型对象 `Response<T>`
- **分包策略**: 按业务领域分包 (`dataCollector`, `modelService` 等)，而非按层分包。
- **错误处理**: 使用全局异常处理 (`@ControllerAdvice`)，禁止吞掉异常。

#### 定时任务规范（强制）

1. **统一任务调度中心**：所有定时任务必须通过 `tradingExecutor.job` 模块中的统一调度体系完成，禁止在业务类中直接使用 `@Scheduled` 注解。
2. **任务注册方式**：
   - 在 `job_config` 表中配置任务的 `jobName` / `beanName` / `methodName` / `cronExpression`；
   - 或在 `JobBootstrap` 中添加默认任务配置，由 `JobSchedulerService` 统一加载。
3. **任务执行入口**：
   - 任务执行逻辑应放在独立的 `Scheduler` / `Job` 类中（如 `DataSyncScheduler`、`ModelTrainingRecordSyncJob`）；
   - 方法为无参 `public void methodName()`，供调度中心通过反射调用。
4. **Agent 约束**：后续新增或修改任何后台任务时：
   - **禁止**在任意类上新增 `@Scheduled`；
   - 必须通过 `JobConfig + JobSchedulerService` 的方式接入统一调度体系。

#### 接口日志规范（强制）

1. **全局入口日志**：所有 `@RestController` 接口方法必须在入口打印一行访问日志，用于排查“接口未返回/超时/参数异常”等问题。  
   - 已通过全局 AOP 切面 `com.stock.logging.ApiLoggingAspect` 统一实现，记录控制器类名、方法名和关键参数。
2. **控制器级业务日志**：对于关键接口（如分页查询、大批量操作、模型训练/同步等），在 Controller 方法内 **显式增加入口业务日志**，说明：
   - 调用的业务含义（例如“分页查询模型训练记录”）
   - 关键请求参数（如 `keyword`、`current`、`pageSize` 等）
3. **Agent 约束**：后续新增任何后端接口时，必须遵守：
   - Controller 类使用 `@Slf4j`；
   - 在每个 `public` 接口方法入口增加一条 `log.info(...)` 业务日志（除全局切面外的补充信息），便于在日志中快速定位该接口的调用与入参。

#### 接口入参与返回值规范（强制）

1. **统一使用 DTO 封装**：
   - 所有 `@RestController` 方法的请求体入参和响应体返回值必须使用明确的 DTO/VO 类进行封装；
   - **禁止**在接口层直接使用 `Map`、`List<Map>`、原始 `Object` 作为入参或返回值类型（包括 `ResponseEntity<Map<...>>`、`@RequestBody Map` 等）。
2. **请求参数规范**：
   - 复杂请求体必须定义独立的 `*Request` / `*Param` DTO 类（如 `SentimentTrainingRequest`）；
   - 简单查询参数可继续使用 `@RequestParam` 标量类型（如 `String keyword`、`int page`），但一旦字段超过 3 个，建议封装为请求 DTO。
3. **响应结果规范**：
   - 统一使用封装的响应泛型（如 `ApiResponse<T>` / `PageResult<T>`），`T` 必须是具体的 DTO/VO 类；
   - 对于列表、分页等结构，DTO 内部再包含集合字段，而不是在 Controller 中返回原始 `Map` 拼装结构。
4. **Agent 约束**：
   - 后续新增或修改任何接口时，如果发现使用了 `Map` 作为接口层入参/出参，必须先补齐/重构为对应的 DTO，再进行业务实现；
   - 如需在内部使用 `Map` 做临时计算，必须限制在 Service 层或更下游，且最终对外接口仍然是 DTO。

#### 接口路径规范（强制）

1. **路径命名风格**：
   - 统一使用**小写驼峰式单词组合**（lower camel words），单词之间采用连字符 `-` 分隔，例如：
     - `/api/stockData`、`/api/stock-data` 统一收敛为：`/api/stockData` 或 `/api/stock-data` 中的一种，全项目保持一致；
     - 推荐对资源名使用小写+连字符（REST 风格），如：`/api/model-training-records`、`/api/stock-data`。
2. **版本化与前缀**：
   - 所有业务接口必须以 `/api/` 为统一前缀，后续如需版本化可扩展为 `/api/v1/...`。
3. **避免混用风格**：
   - 禁止在同一项目中同时存在 `/api/stockInfo` 与 `/api/stock-info` 这种混用情况；
   - 新增接口时必须对齐已有约定的命名风格，若需调整旧路径，须保留一段时间的兼容映射。

#### 接口路径参数位置规范（强制）

1. **路径参数放在末尾**：
   - 所有包含路径参数的接口，必须将路径参数段放在 URL 的最后位置；
   - 例如：`/api/jobs/{id}/status` 必须重构为 `/api/jobs/status/{id}`，`/api/modelInfo/{code}/details` 必须重构为 `/api/modelInfo/details/{code}`。
2. **Agent 约束**：
   - 后续新增任何带路径参数的接口时，禁止在参数段后再追加其它路径片段；
   - 如发现旧接口不符合约定，需在不破坏前端的前提下，通过增加新路径 + 兼容映射的方式逐步迁移。

#### DTO 字段注释规范（强制）

1. **字段级别注释必填**：
   - 所有 DTO/VO 类中的每一个字段，必须在字段声明前添加 Javadoc 或行级注释，清晰说明字段含义、单位、取值范围（如适用）；
   - 示例：
     ```java
     /**
      * 股票代码，如 600519
      */
     private String stockCode;
     ```
2. **适用范围**：
   - 新增 DTO 必须严格按照规范编写字段注释；
   - 修改已有 DTO 时，如发现缺失字段注释，需一并补齐。

#### 类注释与作者信息规范（强制）

1. **统一作者信息**：
   - 所有新建或修改的 Java 类，类级 Javadoc 注释中的作者统一使用 `@author mwangli`
   - 不再使用其他作者标识。
2. **创建时间必填**：
   - 类级 Javadoc 必须包含创建时间字段，例如：
     ```java
     /**
      * 股票数据服务类
      *
      * @author mwangli
      * @since 2026-03-10
      */
     ```
3. **Agent 约束**：
   - 新增类时必须按照上述模板补充类注释；

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mwangli/stock-trading](https://github.com/mwangli/stock-trading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
