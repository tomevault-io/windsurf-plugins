---
trigger: always_on
description: 这是一个基于 AI 模型的双色球彩票预测系统，展示多个 AI 模型（GPT-5、Claude 4.5、Gemini 2.5、DeepSeek R1）对双色球开奖号码的预测，并提供历史预测准确率对比功能。
---

# 双色球 AI 预测系统

## 项目概述

这是一个基于 AI 模型的双色球彩票预测系统，展示多个 AI 模型（GPT-5、Claude 4.5、Gemini 2.5、DeepSeek R1）对双色球开奖号码的预测，并提供历史预测准确率对比功能。

**核心特性**:
- 🤖 多 AI 模型预测展示
- 📊 历史开奖数据查询
- 🎯 预测命中率统计
- ⏰ 自动更新开奖数据（GitHub Actions）
- 🔮 下期开奖信息展示

**双色球规则**:
- 红球：从 01-33 中选择 6 个号码
- 蓝球：从 01-16 中选择 1 个号码
- 开奖时间：每周二、四、日 21:15

---

## 项目结构

```
Double-Color-Ball-AI/
├── index.html                    # 主页面
├── css/
│   └── style.css                 # 样式文件（支持深色/浅色主题）
├── js/
│   ├── app.js                    # 主应用逻辑
│   ├── components.js             # UI 组件
│   └── data-loader.js            # 数据加载模块
├── data/                         # 前端数据文件
│   ├── lottery_history.json      # 历史开奖数据 + 下期开奖信息
│   ├── ai_predictions.json       # 当前 AI 预测（未开奖期号）
│   └── predictions_history.json  # 历史预测对比数据（已开奖期号）
├── fetch_history/                # 数据爬取脚本
│   ├── fetch_lottery_history.py  # 爬虫脚本（自动同步到 data/）
│   ├── lottery_data.json         # 爬虫原始数据
│   └── lottery_data_backup_*.json # 自动备份文件（不提交到 Git）
├── .github/workflows/
│   └── update-lottery-data.yml   # GitHub Actions 自动更新配置
├── add_gpt5_prediction.py        # 辅助脚本：添加历史预测
├── start_server.sh / .bat        # 本地开发服务器启动脚本
├── vercel.json                   # Vercel 部署配置
├── DATA_UPDATE_GUIDE.md          # 数据更新指南
└── .gitignore                    # Git 忽略规则（排除备份文件）
```

---

## 核心数据文件

### 1. `data/lottery_history.json`
**用途**: 网页前端使用的历史开奖数据

**数据结构**:
```json
{
  "last_updated": "2025-10-22T20:39:53Z",
  "data": [
    {
      "period": "25121",
      "date": "2025-10-21",
      "red_balls": ["06", "08", "10", "25", "29", "30"],
      "blue_ball": "08"
    }
  ],
  "next_draw": {
    "next_period": "25122",
    "next_date": "2025-10-23",
    "next_date_display": "2025年10月23日",
    "weekday": "周四",
    "draw_time": "21:15"
  }
}
```

**更新方式**:
- 自动：GitHub Actions 每天 22:00 自动运行爬虫更新
- 手动：运行 `cd fetch_history && python3 fetch_lottery_history.py`

---

### 2. `data/ai_predictions.json`
**用途**: 当前 AI 模型对未开奖期号的预测

**数据结构**:
```json
{
  "prediction_date": "2025-10-23",
  "target_period": "25122",
  "models": [
    {
      "model_id": "SSB-Team-001",
      "model_name": "GPT-5",
      "predictions": [
        {
          "group_id": 1,
          "strategy": "热号追随者",
          "red_balls": ["02", "09", "17", "25", "31", "33"],
          "blue_ball": "02",
          "description": "策略描述..."
        }
        // 每个模型 5 组预测
      ]
    }
    // 4 个 AI 模型
  ]
}
```

**更新时机**: 当 `target_period` 开奖后，需要手动更新为下一期的预测

---

### 3. `data/predictions_history.json`
**用途**: 已开奖期号的历史预测和命中结果

**数据结构**:
```json
{
  "predictions_history": [
    {
      "prediction_date": "2025-10-21",
      "target_period": "25121",
      "actual_result": {
        "period": "25121",
        "date": "2025-10-21",
        "red_balls": ["06", "08", "10", "25", "29", "30"],
        "blue_ball": "08"
      },
      "models": [
        {
          "model_id": "SSB-Team-001",
          "model_name": "GPT-5",
          "predictions": [
            {
              "group_id": 1,
              "strategy": "热号追随者",
              "red_balls": ["02", "09", "17", "25", "31", "33"],
              "blue_ball": "02",
              "description": "...",
              "hit_result": {
                "red_hits": ["25"],
                "red_hit_count": 1,
                "blue_hit": false,
                "total_hits": 1
              }
            }
          ],
          "best_group": 2,
          "best_hit_count": 2
        }
      ]
    }
  ]
}
```

**更新方式**: 当期号开奖后，运行脚本计算命中结果并添加到历史

---

## 关键脚本

### `fetch_history/fetch_lottery_history.py`
**功能**:
1. 从 500.com 爬取最新双色球开奖数据
2. 与现有数据合并去重
3. 自动创建带时间戳的备份文件
4. **自动同步到 `../data/lottery_history.json`**（网页数据）
5. **自动计算下期开奖信息**（基于周二、四、日规律）

**运行方式**:
```bash
cd fetch_history
python3 fetch_lottery_history.py
```

**依赖**:
- `requests` - HTTP 请求
- `beautifulsoup4` - HTML 解析

---

### `add_gpt5_prediction.py`
**功能**: 将某期的 AI 预测添加到历史记录并计算命中结果

**使用场景**: 当某期开奖后，需要将预测数据从 `ai_predictions.json` 移到 `predictions_history.json`

**自动功能**:
- 计算红球命中数和蓝球命中情况
- 计算每个模型的最佳预测组
- 添加到历史记录顶部

---

## GitHub Actions 自动化

### 工作流文件
`.github/workflows/update-lottery-data.yml`

### 触发时机
- **定时**: 每天 UTC 14:00（北京时间 22:00）
- **手动**: GitHub Actions 页面点击 "Run workflow"

### 执行流程
1. 安装 Python 和依赖（requests, beautifulsoup4）
2. 运行 `fetch_lottery_history.py` 爬取数据
3. 检测 `data/` 目录是否有变更
4. 如有变更，自动提交并推送到仓库
5. Vercel 监听到仓库更新，自动重新部署

### 权限配置
确保仓库设置中启用了工作流写入权限：
- Settings > Actions > General > Workflow permissions
- 选择 **Read and write permissions**

---

## 前端架构

### 页面结构
- **顶部导航**: 标题、刷新按钮、主题切换
- **Tab 切换**:
  - AI 预测：下期开奖卡片 + 最新开奖结果 + 预测状态 + 多模型预测
  - 预测对比：历史预测命中率对比
  - 历史开奖：历史开奖记录列表

### 主要功能

#### 1. 下期开奖卡片
- 显示下期期号、日期、星期、开奖时间
- 智能检测是否有 AI 预测：
  - 🟢 已有AI预测
  - 🟡 暂无AI预测

#### 2. 预测状态
- 比较 `ai_predictions.json` 的 `target_period` 和最新开奖期号
- 显示两种状态：
  - 🔮 **未开奖**: target_period > 最新期号
  - ✅ **已开奖**: target_period ≤ 最新期号

#### 3. 模型选择器
- 下拉菜单切换不同 AI 模型
- 显示该模型的 5 组预测方案

#### 4. 历史预测对比
- 按期号分组显示所有模型的历史预测
- 高亮显示命中的号码
- 显示准确率徽章（优秀/良好/一般/较差）
- 标记每个模型的最佳预测组

### 主题切换
- 支持浅色/深色模式
- 使用 CSS 变量实现
- 状态保存在 localStorage

---

## 数据更新工作流

### 场景 1: 新期号开奖
1. **自动**: GitHub Actions 在 22:00 自动运行爬虫
2. **自动**: 更新 `lottery_history.json` 和 `next_draw` 信息
3. **自动**: 提交并部署到 Vercel
4. **手动**: 运行辅助脚本将旧预测移到历史：
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sinyu1012/Double-Color-Ball-AI](https://github.com/sinyu1012/Double-Color-Ball-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
