---
trigger: always_on
description: 本文件为 Codex (Codex.ai/code) 提供代码库工作指引。
---

# AGENTS.md

本文件为 Codex (Codex.ai/code) 提供代码库工作指引。

## 项目概述

洛克王国 PvP 辅助工具 — 实时战斗分析与建议工具。被动监听游戏 8195 端口的网络流量，解密自定义 BE21 协议，跟踪实时战斗状态，在 PvP 对战中提供可操作建议（属性克制、威胁评估、阵容搭配）。工具纯粹被动——只读取流量，不会向游戏发送任何数据包。

## 运行环境

- **生产运行环境**：Windows — 工具和游戏都在 Windows 上运行
- **开发环境**：可能在 macOS 上开发，但**所有代码必须以 Windows 为基准**
- **注意事项**：使用 Windows 兼容的 API、路径和命令。Scapy 抓包在 Windows 上需要安装 Npcap。使用 `py`（不是 `python`）作为 Python 启动器。功能完成前必须在 Windows 上测试。

## 技术栈

- **后端**：Python 3.9+, FastAPI, Scapy, PyCryptodome, uvicorn
- **Python 启动器**：Windows 上使用 `py`（不是 `python`）— `python` 不在 PATH 中
- **前端**：React 19, TypeScript, Ant Design 6, Zustand, Vite 8, React Router 7
- **协议**：自定义 BE21 二进制帧格式，AES-128-CBC 加密，类 protobuf 消息载荷

## 命令

### 后端
```bash
py -m src.main                   # 启动 FastAPI 服务 :18731（支持热重载）
pytest                          # 运行所有测试
pytest tests/test_crypto.py     # 运行单个测试文件
pytest -k "test_name"           # 按名称模式匹配运行测试
```

### 后端自闭环回放
```bash
py -m scripts.replay_headless --session battle_session_1       # 文本摘要（事件、预测、hook 建议）
py -m scripts.replay_headless --session battle_session_1 --json  # 完整 JSON 输出（写入 tmp/）
py -m scripts.replay_headless --round 7                        # 在第 7 回合停止
py -m scripts.generate_battle_report                           # 生成文本报告（docs/battle_report.txt）
py -m scripts.generate_battle_report --json                    # 输出完整 JSON 到 stdout
py -m scripts.unpack_battle_report path\to\battle.raco-report --verify  # 导入 .raco-report 并验证回放
py -m scripts.replay_to_frontend --delay 80 --session battle_session_1  # 推送回放到前端 WebSocket
py -m scripts.replay_to_frontend --delay 80 --session battle_session_1 --round 7  # 回放到第 7 回合
```

### 战斗包提取

详细文档见 [`docs/extract_battle.md`](docs/extract_battle.md)。

```bash
py -m scripts.extract_battle --session <id>                 # 列出战斗
py -m scripts.extract_battle --session <id> --extract 1     # 提取第 1 场
py -m scripts.extract_battle --session <id> --extract all   # 提取所有
```

### 前端（在 `web/` 目录下）
```bash
npm run dev                     # Vite 开发服务器 :18732
npm run build                   # TypeScript 编译 + Vite 生产构建
npm run lint                    # ESLint 检查
```

## 开发工作流

**每次开发任务完成后必须执行** — 按顺序完成以下步骤：

### 1. 运行完整测试套件
```bash
pytest
```
所有测试必须通过才能继续。先修复失败测试再进行下一步。

### 2. 启动后端和前端
```bash
# 终端 1 — 后端
py -m src.main

# 终端 2 — 前端
cd web && npm run dev
```

### 3. MCP 战斗回放验证（完整的前后端回放验证）

当要求进行"完整的前后端回放验证"时，必须使用 MCP Chrome DevTools 工具进行自动化验证，而非手动操作浏览器。

**步骤：**

1. **导航到战斗页面** — 使用 `mcp__chrome-devtools__navigate_page` 打开 `http://localhost:18732/battle`
2. **建立 WebSocket 连接** — 使用 `mcp__chrome-devtools__take_snapshot` 获取页面快照，找到"连接战斗"按钮的 uid，使用 `mcp__chrome-devtools__click` 点击连接
3. **确认连接成功** — 使用 `mcp__chrome-devtools__take_snapshot` 验证连接状态（如按钮文本变化或状态提示）
4. **运行回放脚本** — 使用 Bash 后台运行：
   ```bash
   # 完整回放
   py -m scripts.replay_to_frontend --delay 80 --session battle_session_1

   # 回放到指定回合（如 R7）停止，用于测试中间状态
   py -m scripts.replay_to_frontend --delay 80 --session battle_session_1 --round 7
   ```
5. **等待回放完成** — 等待脚本执行完毕后，再等待 10 秒缓冲，确保前端完成所有渲染
6. **截图检查页面状态** — 使用 `mcp__chrome-devtools__take_screenshot` 截取页面截图，观察：
   - HP、能量、buff 状态更新是否正确
   - 宠物切换是否正确显示
   - 战斗事件时间线是否正常渲染
   - 属性克制和 counter-pick 建议是否正确显示
7. **检查浏览器日志** — 使用 `mcp__chrome-devtools__list_console_messages` 检查是否有 JS 错误或异常
8. **检查后端日志** — 查看后端控制台输出是否有异常或报错
9. **汇报结果** — 总结验证结果，如有问题则定位并修复

如果发现任何异常，必须调查并修复后才能标记任务完成。

### 4. 截图清理

**审查后立即删除截图文件。** 通过 MCP `take_screenshot` 或其他方式截图后，分析完毕即删除。不要在项目目录中遗留 `.png`/`.jpg`/`.jpeg` 截图文件。此规则适用于主 agent 和子 agent。

### 5. 后端自闭环回放验证

当要求进行"后端自闭环验证"时，使用 `BattleReplayRunner` 进行纯后端验证，无需启动服务器或前端。

**步骤：**

1. **运行 headless 回放** — 使用 Bash 运行：
   ```bash
   py -m scripts.replay_headless --session battle_session_1
   ```
2. **检查输出完整性** — 验证输出包含：
   - 每回合的格式化事件（skill_cast, damage, defeat, effect_apply 等）
   - 每回合的伤害预测（技能名称、预期伤害、效果标签、KO 标记）
   - 建议（低血量、低能量、击杀提示等）
   - Hook 建议（换宠建议、能量监控、对手行为分析）
   - 最终状态（双方阵容 HP）
3. **运行 JSON 输出对比** — 使用 `--json` 生成结构化数据进行字段级验证
4. **运行相关测试** — `pytest tests/test_replay_runner.py -v` 确保所有回放测试通过

## 架构

系统是一个分层管线，在抓包、解析、分析、展示之间有清晰的边界：

```
网络流量（端口 8195）
  │
  ▼
capture/sniffer.py ── Scapy AsyncSniffer，编排整个管线
  │
  ├── capture/key_capture.py ── 从 ACK 包提取 AES 会话密钥
  ├── capture/reassembly.py ── TCP 流重组为有序流
  ├── capture/frame.py ── BE21 帧解析（头部 + 载荷提取）
  ├── capture/crypto.py ── AES-128-CBC 解密加密载荷
  │
  ▼
protocol/
  ├── proto_core.py ── Protobuf 解析器、TGCP 传输（4 种格式）、宠物/状态提取、游戏常量
  ├── opcodes.py ── 基于装饰器的 opcode/内部消息注册与分发
  ├── battle.py ── 战斗专用提取（双路径：schema 优先 + 原始字段回退）
  │
  ▼
analysis/
  ├── constants.py ── 共享 opcode 常量、OPCODE_LABELS、SDT_TO_TYPE 重导出（单一数据源）
  ├── pet_info.py ── PetInfo 构建工厂（from_wrapper/from_change_pet → to_dict），统一宠物字典构建，包含 base_speed（来自 battle_stats[5]）
  ├── battle_state.py ── 实时战斗状态机（HP、能量、buff、回合追踪）
  ├── battle_processor.py ── 纯同步事件处理器（状态 + 格式化 + 伤害 + hooks），BattleManager 和 ReplayRunner 共用
  ├── battle_advisor.py ── 战斗分析协调器（技能分析 + 伤害预测 + 状态建议）
  ├── damage_calc.py ── 伤害计算引擎，4 阶段 hook 管线
  ├── innate_hooks.py ── 天赋技能伤害 hooks（连击/属性/类型/威力修改）
  ├── event_formatter.py ── 协议事件 → UI 格式化事件
  ├── replay_runner.py ── 无头回放器（无 FastAPI/WebSocket），生成 ReplayResult 和逐事件快照
  ├── hook_registry.py ── 可扩展分析 hook 系统（基于 ABC，生命周期感知）
  ├── hooks/
  │   ├── __init__.py ── 默认 hook 工厂
  │   ├── opponent_tracker.py ── 对手技能/换宠模式追踪
  │   ├── energy_monitor.py ── 能量监控与攻击窗口检测

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [innovationb1ue/rocom-helper](https://github.com/innovationb1ue/rocom-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
