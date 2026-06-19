---
trigger: always_on
description: 金蝶云苍穹开发主 Skill，优先复用 kd-cd-cosmic-commons 封装。适用于插件开发、单据/列表/表单逻辑、操作服务、BOTP 转换、后台视图打开、附件处理、DynamicObject 与元数据处理、弹性域解析及 OpenAPI 集成。默认优先使用仓库封装；在涉及原生插件事件、SDK API、方法签名或封装未覆盖场景时，使用内置脚本进行查询。
---


# 苍穹开发

默认按"封装优先，原生兜底"工作，避免在仓库已封装的场景里退回到 BOS 原生低层 API。

## 最短决策路径

1. 先判断插件类型或能力类别（查下方决策矩阵）。
2. 先读对应 `references/*.md`，确认事件边界与适用场景。
3. 再读对应 `assets/*.java` 模板，沿用已有方法签名和骨架。小场景可直接用 `assets/snippets/*.java`。
4. 字段不确定先查 `cosmic-form-metadata.py`，SDK 签名不确定先查 `cosmic-api-knowledge.py`。
5. 只有"插件类型 + 事件方法 + 字段/签名"都确认后，才开始生成代码。
6. **代码生成后，必须执行 `cosmic-post-lint.py` 自动校验**；若存在 ERROR 级问题须立即修复并重新校验直到通过。

## 快速决策矩阵

| 需求关键词 | 插件类型          | 封装文档 (先读) | 原生文档 (兜底) | 模板文件 |
|---|---------------|---|---|---|
| 表单 UI / 字段联动 / 控件交互 | 表单插件          | [plugin-base.md](references/adv/plugin-base.md) | [plugin-form.md](references/base/plugin/plugin-form.md) | [FormPluginTemplate.java](assets/FormPluginTemplate.java) |
| 单据 UI / 审核提交按钮 | 单据插件          | [plugin-base.md](references/adv/plugin-base.md) | [plugin-bill.md](references/base/plugin/plugin-bill.md) | [BillPlugInTemplate.java](assets/BillPlugInTemplate.java) |
| 列表 / 多选操作 / 批量 | 列表插件          | [plugin-base.md](references/adv/plugin-base.md) | [plugin-list.md](references/base/plugin/plugin-list.md) | [ListPluginTemplate.java](assets/ListPluginTemplate.java) |
| 左树右表（单据） | 单据树列表插件（列表）   | [plugin-base.md](references/adv/plugin-base.md) | [plugin-tree-list.md](references/base/plugin/plugin-tree-list.md) | [TreeListPluginTemplate.java](assets/TreeListPluginTemplate.java) |
| 左树右表（基础资料） | 基础资料树列表插件（列表） | [plugin-base.md](references/adv/plugin-base.md) | [plugin-tree-list.md](references/base/plugin/plugin-tree-list.md) | [StandardTreeListPluginTemplate.java](assets/StandardTreeListPluginTemplate.java) |
| 审核 / 保存 / 状态流转 / 校验 | 操作插件          | [operate-chain.md](references/adv/operate-chain.md) | [plugin-operation.md](references/base/plugin/plugin-operation.md) | [OpPluginTemplate.java](assets/OpPluginTemplate.java) |
| 下推 / 选单 / 转换 | 转换插件          | [botp-convert.md](references/adv/botp-convert.md) | [plugin-botp.md](references/base/plugin/plugin-botp.md) | [ConvertPlugInTemplate.java](assets/ConvertPlugInTemplate.java) |
| 反写 / 关联更新 | 反写插件          | [botp-convert.md](references/adv/botp-convert.md) | [plugin-writeback.md](references/base/plugin/plugin-writeback.md) | [WriteBackPlugInTemplate.java](assets/WriteBackPlugInTemplate.java) |
| 报表 / 数据分析 | 报表插件          | — | [plugin-report-form.md](references/base/plugin/plugin-report-form.md) | [ReportFormPluginTemplate.java](assets/ReportFormPluginTemplate.java) |
| 报表取数 | 报表取数插件        | — | [plugin-report-data.md](references/base/plugin/plugin-report-data.md) | [ReportListDataPluginTemplate.java](assets/ReportListDataPluginTemplate.java) |
| 打印 / 套打 | 打印插件          | — | [plugin-print.md](references/base/plugin/plugin-print.md) | [PrintPluginTemplate.java](assets/PrintPluginTemplate.java) |
| OpenAPI / 外部集成 | OpenAPI 控制器   | — | [plugin-openapi.md](references/base/plugin/plugin-openapi.md) | [OpenApiControllerTemplate.java](assets/OpenApiControllerTemplate.java) |
| 后台任务 / 定时 | 后台任务          | — | [plugin-task.md](references/base/plugin/plugin-task.md) | [TaskTemplate.java](assets/TaskTemplate.java) |
| 工作流 / 审批流 | 工作流插件         | — | [plugin-workflow.md](references/base/plugin/plugin-workflow.md) | [IWorkflowPluginTemplate.java](assets/IWorkflowPluginTemplate.java) |
| 导入 / 批量导入 | 导入插件          | — | [plugin-import.md](references/base/plugin/plugin-import.md) | [BatchImportPluginTemplate.java](assets/BatchImportPluginTemplate.java) |

## 能力封装路由（按需加载，只读相关的 1-2 个文档）

- **IF** 涉及保存/提交/审核/回滚 → 读 [operate-chain.md](references/adv/operate-chain.md)
- **IF** 涉及下推/选单/来源追踪 → 读 [botp-convert.md](references/adv/botp-convert.md)
- **IF** 涉及查询/聚合/DataSet → 读 [query-dataset.md](references/adv/query-dataset.md)
- **IF** 涉及后台打开表单/列表 → 读 [view-handler.md](references/adv/view-handler.md)
- **IF** 涉及表单控件/UI消息/元数据读取 → 读 [form-utils.md](references/adv/form-utils.md)
- **IF** 涉及 DynamicObject 安全取值/序列化 → 读 [dynamic-object.md](references/adv/dynamic-object.md)
- **IF** 涉及实体元数据/字段路径/DBRoute → 读 [entity-metadata.md](references/adv/entity-metadata.md)
- **IF** 涉及弹性域字段/值解析 → 读 [flex-prop.md](references/adv/flex-prop.md)
- **IF** 涉及附件上传/下载/复制 → 读 [attachment-api.md](references/adv/attachment-api.md)
- **IF** 涉及跨线程身份/上下文恢复 → 读 [request-context.md](references/adv/request-context.md)

## 原生 SDK 兜底路由（仅在封装层不够用时）

- ORM/QFilter/KSQL → [sdk-orm-access.md](references/base/sdk/sdk-orm-access.md)
- 内存计算/DataSet → [sdk-algo.md](references/base/sdk/sdk-algo.md)
- 公共服务/ServiceHelper → [sdk-dynamic-model-svc.md](references/base/sdk/sdk-dynamic-model-svc.md)
- DynamicObject → [sdk-dynamic-object.md](references/base/sdk/sdk-dynamic-object.md)
- 元数据/EntityModel → [sdk-entity-model.md](references/base/sdk/sdk-entity-model.md)
- 分布式ID → [sdk-id.md](references/base/sdk/sdk-id.md) | 锁 → [sdk-lock.md](references/base/sdk/sdk-lock.md) | 缓存 → [sdk-cache.md](references/base/sdk/sdk-cache.md) | 事务KDTX → [sdk-tx.md](references/base/sdk/sdk-tx.md)
- 日志 → [sdk-log.md](references/base/sdk/sdk-log.md) | 异常 → [sdk-exception.md](references/base/sdk/sdk-exception.md) | 线程池 → [sdk-threadpool.md](references/base/sdk/sdk-threadpool.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChenZeXian/ok-cosmic](https://github.com/ChenZeXian/ok-cosmic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
