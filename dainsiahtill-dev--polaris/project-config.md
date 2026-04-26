---
trigger: always_on
description: **目标**：把工程交付做成 **可重复、可审计、可回滚、可防御** 的流水线，同时验证 Polaris 在复杂场景下的稳定性。
---

# Polaris Agent 角色规范 v4.0（Codex 专用）

**目标**：把工程交付做成 **可重复、可审计、可回滚、可防御** 的流水线，同时验证 Polaris 在复杂场景下的稳定性。  
**口号**：精准 > 速度；证据 > 声称；最小变更 > 顺手重构；多层防御 > 单点信任。  
**编码要求**：所有文本文件读写必须显式使用 `UTF-8`（包括日志/JSON/Markdown）。
**铁律**：Polaris 是元工具平台，禁止在主仓代码中添加任何目标项目/业务相关代码。

> 本文件为 **Codex 专用治理规范**。若项目内存在更细粒度 `AGENTS.md`，以项目规则优先，但不得弱化本文件的强制门禁。

> 后端任务强制入口（2026-03-22）：  
> 1) `src/backend/AGENTS.md`  
> 2) `src/backend/docs/AGENT_ARCHITECTURE_STANDARD.md`  
> 且必须执行“Cell 复用优先 + KernelOne 底座优先”。

---

## 一、执行栈（固定顺序）

1. **Playwright**（主流程）- 浏览器自动化测试与操作
2. **Computer Use**（视觉兜底）- 无 DOM 场景的视觉操作（已移除 Hybrid 相关旧测试）

---

## 二、角色系统（唐朝官员制度）

Polaris 采用**唐朝官员制度**的多 Agent 治理架构，每个角色有明确职责边界：

| 官职 | 角色 | 职责 | CLI 入口 |
|------|------|------|----------|
| 尚书令 | **PM** | 项目规划、任务拆分、质量门禁 | `scripts/pm/cli.py` |
| 中书令 | **Architect** | 架构设计、技术选型 | `role_agent/architect_cli.py` |
| 工部尚书 | **Chief Engineer** | 技术分析、代码审查、策略制定 | `role_agent/chief_engineer_cli.py` |
| 工部侍郎 | **Director** | 代码执行、文件操作、命令运行 | `scripts/director/cli_thin.py` |
| 门下侍中 | **QA** | 质量审查、测试验证 | Factory/Pipeline 集成 |
| 探子 | **Scout** | 只读代码探索/文档阅读 (sub-agent) | 即将添加 |

### 探子角色 (Scout) - 规划中

- **定位**: 并发只读访问层（sub-agent）
- **核心价值**: 解决单次 LLM 上下文有限问题，支持多路并发读取
- **执行模式**:
  - 探索模式 → 探索目录结构/模块
  - 搜索模式 → 搜索特定内容
  - 总结模式 → 读取并总结文件内容
- **调用方式**: 由 PM/Director 按需调用
- **特点**: 只做读取，不做写入；由调用者自己汇总结果

```bash
# Architect (中书令) - 交互式架构设计
python -m core.polaris_loop.role_agent.architect_cli --mode interactive --workspace .

# Chief Engineer (工部尚书) - 交互式技术分析
python -m core.polaris_loop.role_agent.chief_engineer_cli --mode interactive --workspace .

# 统一角色对话 API
curl -X POST http://127.0.0.1:49977/v2/role/{pm|architect|chief_engineer|director|qa}/chat \
  -d '{"message": "你的问题"}'
```

---

## 二、硬约束

1. **所有文本读写必须显式 UTF-8**
2. **零信任**：任何上游输出都不默认可信，必须二次校验
3. **禁止"声称修复但未复测"**：每次修复后必须复测对应门禁
4. **禁止仅做表层补丁**：必须定位根因并修复
5. **修复范围允许双域**：Polaris 主仓 + C:/Temp 新项目
6. **运行策略为"直到通过"**：不设轮次上限，持续循环，直到所有验收门禁 PASS
7. **只能修改 Polaris**：绝对不能修改目标项目的任何代码

### 后端迁移承载规则

对于 `src/backend` 下的 ACGA 2.0 / Cell 化 / Context Plane / Governance 演进：

- 任何 Agent 开工前必须先读：`src/backend/docs/AGENT_ARCHITECTURE_STANDARD.md`
- 所有 Cell 开发必须先复用已有 Cell 公开能力，所有新开发必须基于 `src/backend/polaris/kernelone/` 能力与契约链路
- 新架构目标实现统一落在 `src/backend/polaris/`
- 其中规范根目录解释为：
  - `bootstrap/` -> `src/backend/polaris/bootstrap/`
  - `delivery/` -> `src/backend/polaris/delivery/`
  - `application/` -> `src/backend/polaris/application/`
  - `domain/` -> `src/backend/polaris/domain/`
  - `kernelone/` -> `src/backend/polaris/kernelone/`
  - `infrastructure/` -> `src/backend/polaris/infrastructure/`
  - `cells/` -> `src/backend/polaris/cells/`
  - `tests/` -> `src/backend/polaris/tests/`
- `src/backend/docs/graph/`、`src/backend/docs/governance/`、`src/backend/docs/templates/` 继续保留在仓库现有位置，作为共享真相与治理资产
- `src/backend/app/`、`src/backend/core/`、`src/backend/api/`、`src/backend/scripts/` 仍属于迁移中的旧根目录；除兼容垫片、转发入口和必要修复外，不应继续承载新的主实现

---

## 三、测试目标：复杂项目验证

### 3.1 复杂项目要求

为验证 Polaris 稳定性，每次测试需生成**足够复杂**的项目：

- **功能复杂度**：至少 3 个以上模块/服务
- **代码规模**：至少 500+ 行代码（前端+后端）
- **依赖复杂度**：至少 3 层依赖关系
- **测试覆盖**：需要单元测试 + 集成测试
- **配置复杂度**：至少包含配置文件、环境变量、构建脚本

### 3.2 推荐测试项目类型

```
1. RESTful API 服务（Express/Koa + 数据库）
2. Web 应用（React + 后端 API）
3. CLI 工具（Node.js/Python）
4. 微服务架构（多个服务通信）
5. 带数据库的完整应用
```

### 3.3 项目复杂度指标

| 指标 | 最低要求 | 验证方式 |
|------|---------|---------|
| 文件数量 | ≥10 个 | `find . -type f | wc -l` |
| 代码行数 | ≥500 行 | `wc -l **/*.{ts,js,py}` |
| 模块数量 | ≥3 个 | 目录结构检查 |
| 配置文件 | ≥3 个 | package.json, tsconfig.json 等 |
| 测试文件 | ≥2 个 | `**/*.test.ts` |

---

## 四、执行顺序（固定）

### A. 环境预检

1. 启动 Polaris（Electron + backend 可达）
2. 通过 `window.polaris.getBackendInfo` 获取 baseUrl 和 token
3. 验证 `/settings`、`/runtime/storage-layout` 可访问
4. 若 workspace 不是目标目录，优先走 UI 切换；若 OS 文件选择器不可自动化，则使用 `/settings` 更新 workspace 并记录"UI例外降级"

### B. 政事堂立项（中书令）

1. 在主界面点击"生成计划"打开"政事堂"
2. 在"圣意目标"填入项目需求（**必须足够复杂**）
3. 点击"发起奏对"，至少 1 轮，直到"廷议状态"显示可拟定条陈或"已齐备"
4. 点击"拟定条陈"，等待预览生成完成
5. 点击"批红 / 用印"，确认 docs 成功写入
6. 校验 docs 已生成且 plan 已同步到 runtime contracts

### C. PM 运行与质量门禁

1. 进入 PM 工作区（enter-pm-workspace），点击单次督办（pm-workspace-run-once）
2. 轮询 `/v2/pm/status`：必须出现 `running=true` 后再回到 `false`
3. 校验 `/state/snapshot`：
   - `tasks` 数量 > 0
   - `completed_task_count` > 0
   - `last_director_status` 非空

4. **PM 质量硬门禁**：
   - 分数 >= 80
   - critical issues = 0
   - 任务必须具备：目标、作用域、可执行步骤、可测验收

5. **提示词穿透检测**（PM 输出与任务合同）：
   - 检测关键词：`you are`, `role`, `system prompt`, `no yapping`, `提示词`, `角色设定`, `<thinking>`, `<tool_call>`
   - 一旦命中视为 P0 失败，立即修复并重跑 PM

### D. Director 执行与工具审计

1. 进入 Director 工作区（enter-director-workspace），点击执行（director-workspace-execute）
2. 轮询 `/v2/director/status`：必须进入 RUNNING 再退出 RUNNING
3. 轮询 `/v2/director/tasks`：必须存在 `metadata.pm_task_id` 关联任务

4. **工具调用审计**：
   - 检查是否存在工具调用证据
   - 检查是否出现 `unauthorized=false`/越权阻断事件
   - 检查是否有危险命令/路径穿越被触发

5. 若工具策略异常、越权或无效调用导致失败，修复根因后重跑 Director

### E. 验收与闭环

1. 检查 `integration_qa` 结果为通过态（目标 `reason=integration_qa_passed`）
2. 若失败，定位失败源头（PM 合同、Director 执行、工具策略、代码实现、测试基线）并修复
3. 修复后先回归失败门禁，再做整链回归（政事堂→PM→Director→QA）

---

## 五、修复循环（直到通过）

1. **收集证据**：UI 截图、trace、renderer/terminal 错误、runtime 事件、状态与结果文件
2. **根因分类**：
   - 配置类
   - 提示词类
   - 任务质量类
   - 工具授权类
   - 代码实现类
   - 测试基线类
3. **生成最小充分修复方案并实施**（不得跳过验证）
4. **重跑失败环节 → 重跑全链路**
5. **仅当所有门禁 PASS 才结束**

---

## 六、最终输出（强审计包，JSON）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dainsiahtill-dev/Polaris](https://github.com/dainsiahtill-dev/Polaris) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
