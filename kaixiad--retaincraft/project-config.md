---
trigger: always_on
description: >
---


# RetainCraft

Evidence-based AI-assisted interactive learning protocol
基于循证学习科学的 AI 辅助互动学习协议

> Combines 5 scientifically validated methods + self-assessment + diagnostic test + customized learning path
> 结合 5 种科学验证方法 + 自我评价 + 摸底考试 + 定制化学习路径

**📦 Source Code (源码)**: [https://github.com/kaixiad/RetainCraft](https://github.com/kaixiad/RetainCraft)

**⚠️ Permissions required (所需权限)**:
This skill requires: file read/write (`~/learn/`), Python script execution, web search (for test questions & fact-checking), and platform-specific reminder scheduling. All data stored locally. No external API calls.
本技能需要：文件读写（`~/learn/`）、Python 脚本执行、网络搜索（出题和事实核查）、平台提醒调度。所有数据本地存储，无外部 API 调用。

> **RetainCraft** by [kaixiad](https://github.com/kaixiad) — 170 unit tests, 14 academic citations, zero dependencies.
> If you find this useful, a ⭐ on GitHub would mean a lot.
**📖 Detailed workflow (详细流程)**: [references/full-workflow.md](references/full-workflow.md)

> ⭐ If this skill helps you, please give a Star on GitHub!
> 如果这个 skill 对你有帮助，欢迎在 GitHub 上给个 Star！

---

## ⚠️ Execution Checklist (执行清单)

> All commands use paths relative to this SKILL.md's directory. 以下所有命令路径相对于本文件所在目录。

**Must read before each learning session (每次学习开始前必须读)**

### Critical Steps (关键执行步骤 - 不可违反)

1. **Must execute after module test (模块测试结束后必须执行)**:
   ```bash
   python3 scripts/srs.py record-test <topic> <total> <correct>
   ```
   Not executing = module test invalid, level not updated.
   不执行此命令 = 模块测试无效，等级不更新。

2. **Feynman Check - L5 required (费曼检验 - L5 必需)**:
   - L5 mastery requires: 2 consecutive module tests >=90% + Feynman check passed
   - L5 精通需要：连续 2 次模块测试答对率 >=90% + 费曼检验通过
   - AI plays "confused student", asks 3 questions
   - AI 助手扮演"不懂的学生"，追问 3 个问题

3. **Scoring Discipline (评分纪律 - 不可违反)**:
   - Scoring criteria announced before test, cannot change during test
   - 评分标准在测试开始前公布，测试过程中不可修改
   - Single question score >=7 = "correct"
   - 单题得分 >=7 分 = 算"答对"

4. **Level-up Restrictions (逐级升级限制)**:
   - Levels can only increase one at a time, no skipping
   - 等级只能逐级升级，不能跳级
   - Each upgrade requires 2 consecutive passes
   - 每次升级需要连续 2 次达标

5. **Ensure learning reminder is active (确保学习提醒已生效)**:
   - Check if a timed learning reminder exists for this user
   - If yes → continue normally
   - If no → trigger Step 0.1 reminder creation flow
   AI MUST verify reminder status before proceeding. Missing reminder = learning risk.
   AI 必须确认提醒状态再继续。无提醒 = 学习中断风险。

6. **Manual reminder if not received (未收到提醒可手动执行)**:
   ```bash
   python3 scripts/srs.py reminder
   ```

7. **Check reminder status (检查提醒状态)**:
   - **OpenClaw**: `python3 scripts/srs.py check-reminder`
   - **Other platforms**: Verify using your platform's native mechanism (see Step 0.1 "Reminder check at session start")

8. **Switch reminder channel (切换提醒渠道 — OpenClaw only)**:
   ```bash
   python3 scripts/srs.py switch-channel
   ```

---

## 📚 Core Methodology (核心方法论 - 循证)

| Method (方法) | Effect Size (效果量) | 执行层 | v1.5.0 目标 | Source (来源) |
|---------------|---------------------|--------|------------|---------------|
| Distributed Practice → 间隔重复 | d=0.85 | 🟢 代码级 | 🟢 | Donoghue & Hattie 2021 |
| Practice Testing → 主动回忆 | d=0.74 | 🟢🟡 混合级 | 🟢🟡 | Donoghue & Hattie 2021 |
| Self-Explanation → 费曼学习法 | d=0.54* | 🔵 AI协议级 | 🔵 | Donoghue & Hattie 2021 |
| Interleaved Practice → 交错练习 | d=0.47 | 🔵 AI协议级 | → 🟢 代码级 | Donoghue & Hattie 2021 |
| Elaborative Interrogation → 因果追问 | d=0.56 | 🔵 AI协议级 | 🔵 | Donoghue & Hattie 2021 |
| AI Tutoring (AI 辅导) | 0.63-1.3 SD | 🟢🟡 混合级 | 🟢🟡 | Kestin et al. 2025 RCT |

> **Note (注)**: *d=0.54 corresponds to "Self Explanation" in original paper, mapped to Feynman technique here.
> *d=0.54 对应原文"自我解释"，此处映射为费曼学习法。
> **Execution Layer (执行层)**: 🟢 代码强制执行 | 🟢🟡 代码框架+AI内容 | 🔵 AI在会话中执行

---

## 🔄 Complete Workflow: Five Steps (完整流程：五步启动)

### Step 0: Learning Assessment (学习意愿评估)
- User completes self-assessment questionnaire (主题、目标、水平、时间、偏好)
- 用户完成自我评估问卷

### Step 0.1: Learning Contract (学习契约)
- Trigger: After Step 0, before Step 1
- 触发时机：Step 0 完成后，Step 1 之前
- AI generates a learning plan based on assessment
- AI 助手根据评估生成学习计划
- User can confirm, modify, or skip
- 用户可以确认、修改或跳过

**Output format (输出格式)**:
```
📅 学习时间
- 每周学习天数：周一到周五（5 天）
- 每天学习时间：晚上 8:00 - 9:00（1 小时）
- 休息日：周六、周日（轻量复习）

📚 学习节奏
- 每个模块预计：3-5 天
- 每天新概念：2-3 个
- 每天复习：根据间隔重复算法（SM-2/FSRS-5）到期情况

🎯 目标
- 目标等级：L4 熟练
- 预计总时长：30 小时
- 预计完成日期：2026-06-15

请确认以上计划，或告诉我需要调整的地方。
你可以：
1. 输入"确认"接受计划
2. 输入"修改"调整学习时间
3. 输入"跳过"使用默认设置
```

**Estimated duration formula (预计时长公式)**:
```
预计总时长 = 模块数 × 每模块平均Phase数 × 每Phase平均时长
预计完成日期 = 当前日期 + 预计总时长 / (每日学习时长 × 每周学习天数)
```

**Scientific basis (科学依据)**: Gollwitzer (1999) - Implementation intentions. Specific plans increase execution rate by d=0.65.

**Reminder creation (提醒创建)**:
After user confirms the learning contract, AI MUST create a timed reminder:
1. Extract learning time from the contract (e.g., "每天 20:00")
2. **OpenClaw**: Run `python3 scripts/srs.py setup-reminder` (auto-detects channel + delivery target)
3. **Other platforms**: Create a reminder using your platform's native mechanism, then tell the user the schedule
4. Tell user: "提醒已设置，每天 XX:XX 会提醒你学习"
5. If creation fails → tell user the fallback plan

**Reminder check at session start (会话开始时提醒检查)**:
Every time a learning session starts, check if the user has a timed reminder.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaixiad/RetainCraft](https://github.com/kaixiad/RetainCraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
