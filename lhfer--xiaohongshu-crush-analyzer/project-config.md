---
trigger: always_on
description: >
---


# 小红书用户深度分析 & 追求策略顾问

## 概述

这个 Skill 有三个能力层次：
1. **画像分析**：深度浏览小红书公开主页，构建多维度社会画像
2. **理想型画像**：基于心理学理论框架，从发布内容和收藏行为中推断 TA 会被什么样的人吸引
3. **追求策略**：基于画像生成个性化的接近方案、话术建议和分阶段路线图

整个流程：**深度数据采集 → 多维画像分析 → 理想型推断 → 追求策略生成 → 可视化报告输出**

## 前置条件

- 需要 Claude in Chrome 浏览器工具
- 用户需提供一个小红书用户主页 URL
- **强烈建议用户在浏览器中已登录小红书**——未登录状态下能看到的内容非常有限（约10条笔记、无法点进详情页、无法看收藏），登录后可以获得 5-10 倍的数据量

第一步永远是提醒用户：「为了获得最好的分析效果，请确保你当前的浏览器已经登录了小红书。」

---

## Phase 1：深度数据采集

数据采集是整个分析的基础。采集分为三大部分：笔记列表扫描、笔记抽样深读、收藏夹全量扫描。

### 1.1 主页基础信息

导航到目标用户主页，等待 3 秒加载完成后截图。采集：

```
├── 昵称、小红书号
├── 个人签名 / 简介（非常重要——往往是自我定位的浓缩）
├── IP 属地
├── 星座/年龄等标签（如有展示）
├── 粉丝数 / 关注数 / 获赞与收藏数
├── 头像风格描述
└── 主页背景图（如有）
```

**确认笔记总量：** 用 `read_page` 或 JavaScript 提取「笔记」tab 旁的数字。记录 `TOTAL_NOTES`。

### 1.2 笔记列表全量扫描

**目标：扫描所有笔记标题和基础信息。** 这是后续分析的索引。

小红书主页使用虚拟滚动（virtual scrolling）——DOM 中只保留当前可见的笔记元素，滚出视窗的会被移除。所以不能滚完再提取，必须**边滚边收集**。

**自动滚动收集器模式（推荐）：**

注入以下 JavaScript 到页面，它会在滚动过程中持续收集数据并自动去重：

```javascript
// 初始化收集器（Map 自动去重）
window._noteCollector = { notes: new Map(), scanCount: 0 };

function collectNotes() {
  const sections = document.querySelectorAll('section.note-item, section[class*="note"]');
  sections.forEach(s => {
    const titleEl = s.querySelector('.title span, a .title span');
    const authorEl = s.querySelector('.author-wrapper .name, .footer .name');
    const likeEl = s.querySelector('.like-wrapper .count, .engagement .count');
    if (titleEl) {
      const title = titleEl.textContent.trim();
      const likes = likeEl ? likeEl.textContent.trim() : '0';
      // 用标题前30字符作为去重 key
      const key = title.substring(0, 30);
      if (!window._noteCollector.notes.has(key)) {
        window._noteCollector.notes.set(key, { title, likes });
      }
    }
  });
  window._noteCollector.scanCount = window._noteCollector.notes.size;
}

// 自动滚动 + 收集
async function autoScrollCollect() {
  window._scrollDone = false;
  let prevCount = 0;
  let sameStreak = 0;
  for (let i = 0; i < 500; i++) {
    collectNotes();
    window.scrollBy(0, 500);
    await new Promise(r => setTimeout(r, 400));
    const current = window._noteCollector.notes.size;
    if (current === prevCount) {
      sameStreak++;
      if (sameStreak > 20) break; // 连续20次无新内容 = 到底了
    } else {
      sameStreak = 0;
    }
    prevCount = current;
  }
  window._scrollDone = true;
}
autoScrollCollect();
```

注入后等待 30-60 秒（取决于笔记总量），然后检查 `window._scrollDone` 和 `window._noteCollector.scanCount`。

**Chrome 浏览器扩展断连处理：** 等待期间 Claude in Chrome 扩展可能断开连接。如果 `tabs_context_mcp` 报错或无法访问 tab，重新调用 `tabs_context_mcp` 获取新的 tab 连接，然后用 JavaScript 检查后台脚本是否已完成（检查 `window._scrollDone`）。

**数据提取：** 收集完成后，分批提取（每批100条避免 JSON 截断）：

```javascript
const arr = Array.from(window._noteCollector.notes.values());
JSON.stringify(arr.slice(0, 100));  // 第1批
// JSON.stringify(arr.slice(100, 200));  // 第2批，以此类推
```

**到底确认：** 如果连续 20 次滚动后笔记数不再增加，判定为已到底。记录实际扫到的总数和覆盖率。

### 1.3 笔记抽样深读（3-5 条精读）

笔记深读的目的不是逐条阅读所有内容（那在大量笔记时不现实），而是**精选有代表性的笔记获取深度信息**。配合已有的全量标题列表，少量精读足以支撑高质量分析。

**选择策略——精准抽样：**
1. **最新 1-2 条**：代表当前状态
2. **互动最高的 1 条**：代表 TA 最受认可的内容
3. **低互动但标题很个人化的 1 条**：往往最真实
4. **最早期的 1 条**（如能找到）：了解变化轨迹

对每条深读笔记，采集：
```
├── 完整正文（用 get_page_text）
├── 所有 #标签
├── 发布日期/时间
├── 点赞、收藏、评论数
├── 图片内容描述（截图观察）
├── 评论区分析（前 20 条评论 + 博主回复）
└── 读完后用浏览器返回主页
```

### 1.4 收藏夹全量扫描（极其重要）

**收藏内容是整个分析中含金量最高的数据。** 一个人发布的内容是「前台表演」（front-stage performance），而收藏的内容是 TA 的「内隐偏好」（implicit preferences）——TA 真正在意的、想反复看的、不方便发但很向往的东西。

**对理想型推断和追求策略来说，收藏夹数据的价值甚至超过笔记本身。**

操作流程：
1. 在主页找到「收藏」tab 并点击
2. 如果收藏夹加锁（私密），标注「收藏夹不可见」并跳过
3. 如果公开，使用与笔记列表相同的**自动滚动收集器模式**全量扫描：

```javascript
// 重置收集器（避免残留笔记 tab 的旧数据）
window._favCollector = { notes: new Map(), scanCount: 0 };

function collectFavs() {
  const sections = document.querySelectorAll('section');
  sections.forEach(s => {
    const titleEl = s.querySelector('.title span');
    const authorEl = s.querySelector('.author-wrapper .name, .footer .name');
    const likeEl = s.querySelector('.like-wrapper .count');
    if (titleEl && authorEl) {
      const title = titleEl.textContent.trim();
      const author = authorEl.textContent.trim();
      const likes = likeEl ? likeEl.textContent.trim() : '0';
      const key = title.substring(0, 30) + '|' + author;
      if (!window._favCollector.notes.has(key)) {
        window._favCollector.notes.set(key, { t: title, a: author, l: likes });
      }
    }
  });
  window._favCollector.scanCount = window._favCollector.notes.size;
}

async function autoScrollFavs() {
  window._favScrollDone = false;
  // 先滚到顶部确保从头开始
  window.scrollTo(0, 0);
  await new Promise(r => setTimeout(r, 1000));
  let prevCount = 0;
  let sameStreak = 0;
  for (let i = 0; i < 500; i++) {
    collectFavs();
    window.scrollBy(0, 500);
    await new Promise(r => setTimeout(r, 350));
    const current = window._favCollector.notes.size;
    if (current === prevCount) {
      sameStreak++;
      if (sameStreak > 25) break;
    } else {
      sameStreak = 0;
    }
    prevCount = current;
  }
  window._favScrollDone = true;
}
autoScrollFavs();
```

4. **分批提取**后进行**关键词分类统计**：

```javascript
// 关键词分类模板（根据实际内容调整类别和关键词）
const categories = {
  '美妆/护肤': ['化妆', '护肤', '防晒', '面膜', '口红', '粉底'],
  '穿搭/时尚': ['穿搭', 'ootd', '搭配', '时尚', '衣服'],
  '美食/探店': ['美食', '餐厅', '探店', '甜品', '咖啡'],
  '旅行': ['旅行', '旅游', '攻略', '打卡', '酒店'],
  '健身/运动': ['健身', '运动', '瑜伽', '跑步', '减脂'],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lhfer/xiaohongshu-crush-analyzer](https://github.com/lhfer/xiaohongshu-crush-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
