---
trigger: always_on
description: Copyright 2026 FlagOS Contributors
---

<!--
 Copyright 2026 FlagOS Contributors

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
 -->

# FlagOS 自动化框架 — 项目级指令

> 此文件由 Claude Code 自动加载，提供 Skill 路由、工作流定义和自动决策规则。
> 各步骤的详细执行流程见对应 SKILL.md（通过路由表定位）。

---

## 自动初始化（每次会话启动时检查）

**在执行任何用户任务之前，先静默完成以下初始化**（不需要告知用户）：

权限配置 `settings.local.json` 已由 `run_pipeline.sh` 在启动前自动部署到 `.claude/settings.local.json`。**跳过此步骤，不要尝试创建或复制 settings 文件**。如果是交互式会话（非 pipeline 启动），可检查文件是否存在：

```bash
ls .claude/settings.local.json 2>/dev/null && echo "EXISTS" || echo "MISSING — 请手动执行: mkdir -p .claude && cp settings.local.json .claude/settings.local.json"
```

**注意**：`.claude/` 目录是 Claude Code 的敏感目录，headless 模式下写入会被拦截。pipeline 模式下此文件一定已存在，无需任何操作。

### context.yaml 使用规则（多任务隔离）

- `shared/context.template.yaml` 是模板文件，仅用于 `setup_workspace.sh` 初始化容器，**禁止直接读写**
- 运行时 context 位于容器内 `/flagos-workspace/shared/context.yaml`，每个容器独立，互不干扰
- 读取 context：`docker exec <container> cat /flagos-workspace/shared/context.yaml`
- 写入 context：通过 `docker exec <container>` 在容器内操作
- 宿主机快照：`/mnt/data/flagos-workspace/<model>/config/context_snapshot.yaml`（只读归档，由步骤8和兜底同步写入）
- 宿主机最终状态：`/mnt/data/flagos-workspace/<model>/config/context_final.yaml`（全流程结束时回传）

### 会话恢复检测

初始化完成后，检测是否存在未完成的流程。通过 `docker exec <container> cat /flagos-workspace/shared/context.yaml` 读取容器内 context，如果 `workflow.all_done != true` 且 `container.name` 非空：

1. 运行 `diagnose_failure.py --json` 获取诊断：
   ```bash
   docker exec <container> bash -c "PATH=/opt/conda/bin:\$PATH python3 /flagos-workspace/scripts/diagnose_failure.py --json"
   ```
2. 输出诊断摘要给用户（中断位置、错误原因、恢复建议）
3. 根据诊断结果从中断点恢复（不从头重跑）

如果容器不存在或已停止，提示用户当前状态并询问是否重新开始。

---

## Skill 路由表

| 触发词 | Skill 名称 | SKILL.md 路径 |
|--------|-----------|---------------|
| 容器准备 / prepare container / 环境准备 | flagos-container-preparation | `skills/flagos-container-preparation/SKILL.md` |
| 环境检查 / inspect environment / 服务前检查 | flagos-pre-service-inspection | `skills/flagos-pre-service-inspection/SKILL.md` |
| 启动服务 / start service / 健康检查 | flagos-service-startup | `skills/flagos-service-startup/SKILL.md` |
| 性能测试 / benchmark / vllm bench | flagos-performance-testing | `skills/flagos-performance-testing/SKILL.md` |
| 算子替换 / operator replacement / 算子优化 | flagos-operator-replacement | `skills/flagos-operator-replacement/SKILL.md` |
| 精度评测 / eval correctness / accuracy test / 远端评测 / FlagRelease / flageval / 综合评测 / comprehensive eval / 本地评测 / quick 评测 / evalscope / GPQA | flagos-eval-comprehensive | `skills/flagos-eval-comprehensive/SKILL.md` |
| 日志分析 / analyze logs | flagos-log-analyzer | `skills/flagos-log-analyzer/SKILL.md` |
| 提交 issue / submit issue / report bug / 自动报告 | flagos-issue-reporter | `skills/flagos-issue-reporter/SKILL.md` |
| 组件安装 / install component / 安装 FlagGems / 安装 FlagTree / 升级 FlagGems / flag upgrade | flagos-component-install | `skills/flagos-component-install/SKILL.md` |
| 发布 / 镜像上传 / 镜像打包 / 模型发布 / release / publish / image upload / package image | flagos-release | `skills/flagos-release/SKILL.md` |
| 安装 plugin / install plugin / plugin 安装 / vllm-plugin | flagos-plugin-install | `skills/flagos-plugin-install/SKILL.md` |

---

## 工作流（新模型迁移发布）

**用户提供目标（容器名或镜像地址）+ 模型名后，1-8 全自动执行，零交互。**
**自动识别**：含 `:` 或 `/` 的目标视为镜像地址，否则通过 `docker inspect --type=container` 判断是否为已有容器。模型路径自动搜索，无需手动指定。

```
1 容器准备           → 自动识别容器/镜像 + 模型权重搜索/下载 + 工具部署
2 环境检测           → inspect_env.py 场景分类 + FlagGems 集成分析
3 启服务             → V1(native) + V2(flagos) 启动验证 → 异常自动 issue
4 精度评测           → V1/V2 GPQA Diamond 对比 → 异常自动 issue
5 精度算子调优       → [条件] env_type≠native 且 V2精度下降>5% 时分组排查定位问题算子（最多3轮）
6 性能评测           → V1/V2 4k1k benchmark 对比 → 异常自动 issue
7 性能算子调优       → [条件] env_type≠native 且 ratio<80% 时逐个禁用直到达标
8 自动发布           → 打包 + 上传（统一私有发布，报告注明是否达标）
--- Plugin 验证流程（仅 qualified=true 时触发）---
9  Plugin 安装       → install_plugin.py 安装 vllm-plugin-FL → 失败则 issue + 停止
10 Plugin 启服务     → 以达标算子集 + plugin 模式启动 → 崩溃则 issue + 停止
11 Plugin 精度评测   → 与 V1 基线对比 → 不达标则 issue（继续）
12 Plugin 性能评测   → 与 V1 基线对比 → 不达标则 issue（继续）
13 Plugin 发布       → [不达标]issue + 镜像上传(私有) / [达标]镜像上传 + 更新已发布版本 README
```

执行顺序：1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → [qualified=true] → 9 → 10 → 11 → 12 → 13

**算子累计禁用规则**：5 禁用精度问题算子 → 6 在此基础上测性能 → 7 继续禁用性能问题算子。步骤 10-12 复用步骤 5/7 的最终算子集，不重新调优。各步骤详细流程见对应 SKILL.md 的"编排层指令"章节。

**Plugin 流程特殊规则**：
- 触发条件：步骤 8 完成且 `workflow.qualified=true`，否则步骤 9-13 全部 skipped
- 崩溃停止：步骤 9 安装失败或步骤 10 服务崩溃 → 写 issue → 设 `plugin_workflow.crash_stopped=true` → **停止任务**
- Issue 路由：步骤 9-13 所有 issue 通过 `issue_reporter.py full --type plugin-error --repo flagos-ai/vllm-plugin-FL` 提交（只保存本地文件）
- 不触发算子调优：精度/性能不达标只写 issue，不进入调优流程
- 算子集复用：使用主流程已达标的算子集（含步骤 5/7 的禁用列表），不重新调优，禁止重新检测 GPU
- 镜像 tag：原 date_tag 追加 `-plugin`（如 `202603301143-plugin`）
- Plugin 不达标发布：精度/性能不达标时，先提交 issue，再打包镜像上传 Harbor（私有），不更新 ModelScope/HuggingFace README

### V1/V2/V3 定义


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flagos-ai/FlagRelease](https://github.com/flagos-ai/FlagRelease) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
