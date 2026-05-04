---
trigger: always_on
description: 本文件是仓库级的 agent 工作说明，目标是把 `.cursor/` 下的规则、技能、命令做统一沉淀，供 Codex/Cursor 等代理在进入仓库后直接读取。
---

# AGENTS.md

本文件是仓库级的 agent 工作说明，目标是把 `.cursor/` 下的规则、技能、命令做统一沉淀，供 Codex/Cursor 等代理在进入仓库后直接读取。

## 1. Codex 自动读取 AGENTS.md 的格式与规则（官方核对）

截至 2026-03-11，Codex 对 AGENTS 指令链的核心行为如下：

1. 文件格式：`AGENTS.md` 是标准 Markdown，没有强制字段。
2. 全局层：先看 `~/.codex/AGENTS.override.md`，不存在再看 `~/.codex/AGENTS.md`。该层只取第一个非空文件。
3. 项目层：从项目根目录走到当前工作目录，逐级查找：
   - `AGENTS.override.md`
   - `AGENTS.md`
   - `project_doc_fallback_filenames` 中配置的备用文件名
   每一层目录最多纳入一个文件。
4. 合并顺序：从根到当前目录拼接，越靠近当前目录优先级越高（后拼接，覆盖前面的语义）。
5. 空文件会被忽略。
6. 有大小上限（默认 `project_doc_max_bytes = 32 KiB`），超过上限后不会继续追加指令。
7. 冲突优先级：系统/开发者/用户消息 > AGENTS.md；更深层目录中的 AGENTS > 上层 AGENTS。

## 2. 本仓库说明范围

1. 本文件作用域：仓库根目录及其全部子目录。
2. 当前仓库暂无更深层级的 `AGENTS.md` 或 `AGENTS.override.md`，因此本文件是项目级主说明。
3. 指令来源：`.cursor/rules/*.mdc`、`.cursor/skills/*/SKILL.md`、`.cursor/commands/*.md`。
4. 补充设计文档：`docs/specs/*.md` 中记录已经落地的重要系统设计；配置系统请优先参考 `docs/specs/config-architecture.md`，小故事系统请优先参考 `docs/specs/story-event-system.md`，角色扮演模式请优先参考 `docs/specs/avatar-roleplay-mode.md` 与 `docs/specs/single-choice-unified-framework.md`。
5. 外接控制 API 的服务端当前已经完成一轮模块化收口；优先参考 `docs/specs/external-control-api.md` 中的“当前落地后的服务端模块地图”，不要再把 query builder、command handler、初始化 phase 或宿主装配重新堆回 `src/server/main.py`。

## 3. `.cursor/rules` 沉淀

### 3.1 规则索引

| 规则文件 | 作用范围（globs） | 关键约束摘要 |
|---|---|---|
| `.cursor/rules/action-development.mdc` | `src/classes/action/**/*.py`, `src/classes/mutual_action/**/*.py` | 新动作必须补齐 `ACTION_NAME_ID/DESC_ID/REQUIREMENTS_ID/EMOJI/PARAMS`、生命周期方法、注册装饰器、`__init__.py` 导出和测试。 |
| `.cursor/rules/config-architecture.mdc` | `src/config/**/*.py`, `src/server/**/*.py`, `src/utils/config.py`, `src/utils/llm/**/*.py`, `src/sim/save/**/*.py`, `src/sim/load/**/*.py`, `web/src/stores/setting.ts`, `web/src/api/modules/system.ts`, `web/src/api/modules/llm.ts`, `web/src/components/game/panels/system/**/*.vue`, `web/src/App.vue` | 配置分为只读版本配置、`settings.json/secrets.json` 应用设置、`RunConfig` 本局快照；用户数据统一走 data root；前端设置真源是 `/api/settings`；LLM key 不回传前端；存档需保存 `run_config`。 |
| `.cursor/rules/development-phase.mdc` | `*.py`, `*.vue`, `*.ts`, `*.tsx`, `*.json` | 开发阶段：**原则上不要求**向前/向下兼容，以新代码清晰合理为先；若兼容可**零代价**顺带（如一行 `.get` 且不拖结构）可做，**不得**为兼容付出双轨/分支/牺牲主路径可读性。 |
| `.cursor/rules/docker-persistence.mdc` | `docker-compose.yml`, `deploy/Dockerfile.backend`, `deploy/Dockerfile.frontend`, `deploy/nginx.conf`, `src/config/data_paths.py`, `src/server/main.py`, `src/utils/config.py`, `tests/test_docker_build_contract.py`, `tests/test_data_paths_runtime_contract.py`, `tests/test_docker_runtime_smoke.py`, `tests/test_nginx_proxy_contract.py`, `tests/test_server_binding.py`, `README.md`, `docs/readme/*.md` | Docker 场景统一走 `CWS_DATA_DIR` 持久化；容器不得因无人连接自动退出；frontend 健康检查要反映 backend 代理链路可用；生产镜像只装 runtime 依赖；相关改动必须补 contract/smoke 测试与文档。 |
| `.cursor/rules/external-control-api.mdc` | `src/server/**/*.py`, `web/src/api/**/*.ts`, `web/src/stores/**/*.ts`, `tests/test_api_*.py`, `tests/test_websocket_handlers.py`, `tests/test_init_status_api.py`, `tests/test_game_init_integration.py`, `docs/specs/external-control-api.md`, `README.md` | 外接控制 API 以外部 agent / Claw 接入为第一视角；允许不向前兼容，优先建立 runtime/service/API 分层；稳定接口应按 query/command 分离并统一响应格式；所有 mutation 必须串行化，不得继续扩散 `main.py` 直连 `game_instance` 的写法。 |
| `.cursor/rules/event-system.mdc` | `src/classes/event.py`, `src/classes/event_storage.py`, `src/systems/**/*.py` | 明确 `is_major/is_story` 语义，准确填写 `related_avatars`，统一由模拟器集中入库，查询走分页。 |
| `.cursor/rules/frontend-sound.mdc` | `web/src/**/*.vue|ts|tsx` | 标准按钮默认自动音效，特殊音效用 `v-sound`，禁音用 `data-no-sound`，仅特殊场景允许 `useAudio` 编程式播放。 |
| `.cursor/rules/frontend-typing-error.mdc` | `web/src/**/*.vue|ts|tsx` | DTO 先更新 `types/api.ts`，映射逻辑放 `api/mappers`，避免 `any` 扩散，错误统一 `appError`，Socket 分发在 `socketMessageRouter`。 |
| `.cursor/rules/frontend.mdc` | `web/src/**/*.{vue,ts,js}` | 后端驱动架构、Store 职责边界、根层 UI 采用 `scene + overlay`、启动状态机集中在 `useAppShell`、系统菜单采用壳层/内容层/流程层拆分、Socket 分层、复杂面板逻辑下沉到 composable、懒加载弹窗的 `show` watcher 必须 `immediate`、`shallowRef`/长列表性能策略、容器尺寸统一用 `useElementSize()`；前端 `public` 资源禁止写死站点根绝对路径，必须走 `BASE_URL` helper 或 `src/assets` 模块导入。 |
| `.cursor/rules/i18n-phase1.mdc` | `*.py`, `*.vue`, `*.json`, `*.po` | 当前默认仍是 Phase 1：日常功能开发优先只改 `zh-CN`；但若任务被明确提升为 Phase 2 / 正式多语言补全 / 新增语言接入，则应按 `static/locales/registry.json` 中启用语言统一处理（可包含 `ja-JP`）；i18n 源文件优先维护 `modules/` 与 `game_configs_modules/`，`LC_MESSAGES/*.po` 属于合并产物；`.po` 中 `msgid` 不得直接使用中文。 |
| `.cursor/rules/llm-integration.mdc` | `src/utils/llm/**/*.py`, `src/**/*.py` | 统一走 `src.utils.llm.client`，按场景选 `LLMMode`，重试交给底层，异常捕获 `LLMError/ParseError` 并降级。 |
| `.cursor/rules/roleplay-mode.mdc` | `src/server/**/*.py`, `src/systems/single_choice/**/*.py`, `src/classes/mutual_action/**/*.py`, `src/sim/**/*.py`, `web/src/components/game/**/*.vue`, `web/src/stores/roleplay.ts`, `web/src/api/modules/avatar.ts`, `web/src/types/api.ts`, `tests/test_public_api_v1.py`, `tests/test_single_choice.py`, `tests/test_action_social.py`, `tests/test_mutual_actions.py`, `web/src/__tests__/components/game/**/*.test.ts`, `docs/specs/avatar-roleplay-mode.md`, `docs/specs/single-choice-unified-framework.md` | 默认仍是上帝视角；扮演态是单角色 runtime 接管；只在决策边界暂停；二期优先重构 `single_choice` 为统一有限决策框架；三期对话必须依附 `Conversation` 动作，原始聊天记录不进存档。 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [4thfever/cultivation-world-simulator](https://github.com/4thfever/cultivation-world-simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
