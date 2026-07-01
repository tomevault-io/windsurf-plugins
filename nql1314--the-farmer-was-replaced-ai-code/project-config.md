---
trigger: always_on
description: 本文档详细说明了 The Farmer Was Replaced 游戏中所有排行榜挑战的规则、目标和起始条件。
---

# TFWR 排行榜挑战规则

本文档详细说明了 The Farmer Was Replaced 游戏中所有排行榜挑战的规则、目标和起始条件。

## 📋 排行榜类别概览

### 主要挑战
1. **最快重置** (Fastest_Reset) - 从零开始自动化整个游戏
2. **迷宫** (Maze) - 收获大量金币
3. **恐龙** (Dinosaur) - 收获大量骨头

### 资源挑战（多无人机）
4. **仙人掌** (Cactus)
5. **向日葵** (Sunflowers)
6. **南瓜** (Pumpkins)
7. **木材** (Wood)
8. **胡萝卜** (Carrots)
9. **干草** (Hay)

### 单无人机挑战
10. **迷宫（单）** (Maze_Single)
11. **仙人掌（单）** (Cactus_Single)
12. **向日葵（单）** (Sunflowers_Single)
13. **南瓜（单）** (Pumpkins_Single)
14. **木材（单）** (Wood_Single)
15. **胡萝卜（单）** (Carrots_Single)
16. **干草（单）** (Hay_Single)

---

## 🏆 主要挑战

### 1. 最快重置 (Leaderboards.Fastest_Reset)

#### 挑战目标
从一块空白农田开始，实现游戏完全自动化，直到再次解锁排行榜功能。

#### 起始条件
```python
# 初始状态
unlocks = {}  # 没有任何解锁
items = {}    # 没有任何物品
seed = -1     # 随机种子
```

#### 成功条件
```python
num_unlocked(Unlocks.Leaderboard) > 0
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Fastest_Reset, filename, speedup)
```

#### 等效模拟
```python
unlocks = {}
items = {}
globals = {}
seed = -1  # 负数表示随机种子
simulate(filename, unlocks, items, globals, seed, speedup)
```

#### 策略提示
- 不必解锁全部内容，只需尽快解锁 `Unlocks.Leaderboard`
- 使用 `num_unlocked(unlock) > 0` 检查某项是否已解锁
- 使用 `get_cost(unlock)` 查看解锁成本
- 自动化资源收集以满足解锁需求
- 优先解锁关键功能：循环、函数、感知、扩展

---

### 2. 迷宫 (Leaderboards.Maze)

#### 挑战目标
从解锁全部内容开始，尽快收获 **9,863,168** 份金币。这是重复使用一个 32×32 迷宫 300 次所能获得的金币数量。

#### 起始条件
```python
unlocks = Unlocks  # 所有解锁项（最高级）
items = {
    Items.Weird_Substance: 1000000000,  # 大量奇异物质
    Items.Power: 1000000000              # 大量能量
}
seed = -1
```

#### 成功条件
```python
num_items(Items.Gold) >= 9863168
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Maze, filename, speedup)
```

#### 等效模拟
```python
unlocks = Unlocks
items = {
    Items.Weird_Substance: 1000000000,
    Items.Power: 1000000000
}
globals = {}
seed = -1
simulate(filename, unlocks, items, globals, seed, speedup)
```

#### 策略提示
- 使用 32×32 迷宫（最大尺寸）
- 重复使用迷宫 300 次（在宝藏上继续使用奇异物质）
- 使用高效的迷宫导航算法（BFS/DFS）
- 利用多无人机并行处理多个迷宫
- 确保程序在达到目标时终止

---

### 3. 恐龙 (Leaderboards.Dinosaur)

#### 挑战目标
从解锁全部内容开始，尽快收获 **33,488,928** 根骨头。这是用恐龙尾巴填满一片 32×32 区域所能获得的骨头数量。

#### 起始条件
```python
unlocks = Unlocks  # 所有解锁项（最高级）
items = {
    Items.Cactus: 1000000000,  # 大量仙人掌（购买苹果）
    Items.Power: 1000000000     # 大量能量
}
seed = -1
```

#### 成功条件
```python
num_items(Items.Bone) >= 33488928
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Dinosaur, filename, speedup)
```

#### 等效模拟
```python
unlocks = Unlocks
items = {
    Items.Cactus: 1000000000,
    Items.Power: 1000000000
}
globals = {}
seed = -1
simulate(filename, unlocks, items, globals, seed, speedup)
```

#### 策略提示
- 骨头数量 = 尾巴长度²
- 32×32 = 1024 格，尾巴长度 1024，骨头 = 1024² = 1,048,576
- 但目标是 33,488,928 = 约 32 次完整填满
- 使用哈密尔顿路径填满整个农场
- 优化路径算法减少 tick 消耗
- 注意：只有一架无人机可以装备恐龙帽

---

## 🌾 资源挑战（多无人机）

这些挑战从解锁全部内容开始，目标是尽快收获特定数量的资源。

### 通用起始条件
```python
unlocks = Unlocks  # 所有解锁项（最高级）
items = {
    # 种植该植物所需的资源（大量）
    Items.Power: 1000000000  # 大量能量
}
```

---

### 4. 仙人掌 (Leaderboards.Cactus)

#### 成功条件
```python
num_items(Items.Cactus) >= 33554432
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Cactus, filename, speedup)
```

#### 策略提示
- 使用排序算法（冒泡、插入、快速排序等）
- 排序收割 n 个仙人掌 = n² 产量
- 使用多无人机并行排序和收割
- 最大化单次收割的仙人掌数量

---

### 5. 向日葵 (Leaderboards.Sunflowers)

#### 成功条件
```python
num_items(Items.Power) >= 100000
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Sunflowers, filename, speedup)
```

#### 策略提示
- 种植至少 10 株向日葵
- 收割最大花瓣的那株（5× 奖励）
- 使用多无人机并行管理多个向日葵农场
- 优化花瓣检测和收割时机

---

### 6. 南瓜 (Leaderboards.Pumpkins)

#### 成功条件
```python
num_items(Items.Pumpkin) >= 200000000
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Pumpkins, filename, speedup)
```

#### 策略提示
- 种植大面积南瓜（6×6 或更大）
- 等待全部成熟后合并
- n×n 南瓜：n < 6 时产量 = n³，n ≥ 6 时产量 = n²×6
- 及时替换枯萎南瓜
- 使用多无人机管理多个南瓜田

---

### 7. 木材 (Leaderboards.Wood)

#### 成功条件
```python
num_items(Items.Wood) >= 10000000000
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Wood, filename, speedup)
```

#### 策略提示
- 种植树（每株 5 个木材）
- 使用棋盘格种植避免减速
- 使用浇水和肥料加速生长
- 多无人机并行管理大量树木

---

### 8. 胡萝卜 (Leaderboards.Carrots)

#### 成功条件
```python
num_items(Items.Carrot) >= 2000000000
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Carrots, filename, speedup)
```

#### 策略提示
- 胡萝卜生长时间较长（6 秒）
- 使用浇水（5× 速度）和肥料（-2 秒）
- 混合种植增加产量
- 多无人机并行管理

---

### 9. 干草 (Leaderboards.Hay)

#### 成功条件
```python
num_items(Items.Hay) >= 2000000000
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Hay, filename, speedup)
```

#### 策略提示
- 草生长最快（0.5 秒）
- 让草自然生长，大面积收割
- 多无人机并行遍历收割
- 简单高效，适合新手

---

## 🎯 单无人机挑战

这些挑战限制只能使用**一架无人机**和**8×8 农场**，目标是尽快收获特定数量的资源。

### 通用限制
```python
# 农场大小：8×8
# 无人机数量：1（不能使用 spawn_drone）
# 其他条件与多无人机版本相同
```

---

### 10. 迷宫（单）(Leaderboards.Maze_Single)

#### 成功条件
```python
num_items(Items.Gold) >= 616448
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Maze_Single, filename, speedup)
```

#### 策略提示
- 使用 8×8 迷宫
- 优化单无人机导航算法
- 重复使用迷宫

---

### 11. 仙人掌（单）(Leaderboards.Cactus_Single)

#### 成功条件
```python
num_items(Items.Cactus) >= 131072
```

#### 函数调用
```python
leaderboard_run(Leaderboards.Cactus_Single, filename, speedup)
```

#### 策略提示
- 使用高效排序算法
- 8×8 = 64 个仙人掌，排序收割 = 64² = 4096
- 需要重复多次

---

### 12. 向日葵（单）(Leaderboards.Sunflowers_Single)

#### 成功条件
```python
num_items(Items.Power) >= 10000
```

#### 函数调用
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nql1314/The-Farmer-Was-Replaced-AI-Code](https://github.com/nql1314/The-Farmer-Was-Replaced-AI-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
