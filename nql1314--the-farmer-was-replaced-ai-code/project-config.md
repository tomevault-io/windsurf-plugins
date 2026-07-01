---
trigger: always_on
description: Entities.Grass:      0.5 秒
---

# TFWR 快速参考

## 作物数据

### 生长时间（平均）
```python
Entities.Grass:      0.5 秒
Entities.Dinosaur:   0.2 秒
Entities.Cactus:     1 秒
Entities.Bush:       4 秒
Entities.Pumpkin:    2 秒
Entities.Sunflower:  5 秒
Entities.Carrot:     6 秒
Entities.Tree:       7 秒
```

### 种植成本
```python
Entities.Grass:      免费
Entities.Bush:       免费
Entities.Tree:       免费
Entities.Carrot:     1 木材 + 5 干草
Entities.Pumpkin:    1 胡萝卜
Entities.Sunflower:  25 南瓜
Entities.Cactus:     5 南瓜

# 使用 get_cost() 获取：
cost = get_cost(Entities.Carrot)
# 返回 {Items.Wood: 1, Items.Hay: 5}
```

### 收获产量
```python
# 基础产量（无特殊机制）
Entities.Grass:      1 干草
Entities.Bush:       1 木材
Entities.Tree:       5 木材
Entities.Carrot:     1 胡萝卜
Entities.Pumpkin:    1 南瓜（单个）
Entities.Sunflower:  1 能量（基础）
Entities.Cactus:     1 仙人掌（单个）

# 特殊机制产量：
# 南瓜：n×n 合并 = n³ (n<6) 或 n²×6 (n≥6)
# 仙人掌：排序收割 n 个 = n²
# 向日葵：最大花瓣+10株以上 = 5× 能量
```

### 种植要求
```python
# 需要土壤（Grounds.Soil）：
Entities.Carrot
Entities.Pumpkin
Entities.Sunflower
Entities.Cactus

# 可在草地（Grounds.Grassland）：
Entities.Grass
Entities.Bush
Entities.Tree
```

## 操作成本（Tick）

### 0 Tick（完全免费）
```python
get_tick_count()
get_time()
quick_print()
```

### 1 Tick（查询操作）
```python
# 所有 can_* 函数
can_harvest()
can_move(direction)

# 所有 get_* 函数（除了 get_tick_count 和 get_time）
get_entity_type()
get_ground_type()
get_pos_x()
get_pos_y()
get_world_size()
get_water()
get_companion()

# 其他查询
num_items(item)
num_unlocked(unlock)
measure()
measure(direction)
```

### 200 Tick（执行操作）
```python
move(direction)
harvest()
plant(entity)
till()
swap(direction)
use_item(item)
trade(item, amount)
unlock(unlock_type)
```

### 特殊成本
```python
# spawn_drone()：约 200 ticks
# do_a_flip()：快进时间（成本可变）
# print()：显示文本（暂停执行）
```

## 解锁顺序建议

### 第一阶段：基础编程（必需）
```python
1. Unlocks.Operators      # 运算符
2. Unlocks.Variables      # 变量
3. Unlocks.Loops          # 循环
4. Unlocks.Functions      # 函数
5. Unlocks.Senses         # 感官（获取信息）
```

### 第二阶段：资源扩张
```python
6. Unlocks.Expand         # 扩展农场（3×3）
7. Unlocks.Speed          # 速度升级
8. Unlocks.Lists          # 列表
9. Unlocks.Expand (再次)  # 继续扩展（优先到 6×6）
```

### 第三阶段：高级作物
```python
10. Unlocks.Carrots       # 胡萝卜
11. Unlocks.Pumpkins      # 南瓜（高产）
12. Unlocks.Sunflowers    # 向日葵（能量加速）
13. Unlocks.Watering      # 浇水系统
```

### 第四阶段：优化工具
```python
14. Unlocks.Dictionaries  # 字典
15. Unlocks.Speed (多次)  # 继续升级速度
16. Unlocks.Expand (多次) # 扩展到 10×10
17. Unlocks.Trees         # 树（大量木材）
```

### 第五阶段：高级特性
```python
18. Unlocks.Fertilizer    # 肥料（加速+奇异物质）
19. Unlocks.Mazes         # 迷宫（获取金币）
20. Unlocks.Cactus        # 仙人掌（需要排序）
21. Unlocks.Polyculture   # 混合种植（增产）
```

### 第六阶段：终极工具
```python
22. Unlocks.Megafarm      # 多无人机
23. Unlocks.Debug         # 调试工具
24. Unlocks.Debug_2       # 高级调试
25. Unlocks.Dinosaurs     # 恐龙（获取骨头）
```

### 持续升级
```python
# 这些应该持续升级：
- Unlocks.Speed           # 每级翻倍速度
- Unlocks.Expand          # 增加农场面积
- Unlocks.Watering        # 翻倍水资源生成
- Unlocks.Fertilizer      # 翻倍肥料生成
- Unlocks.Mazes           # 翻倍迷宫产出
```

## 资源获取路径

### 干草（Hay）
```python
# 来源：草
# 策略：让草自然生长，然后收割
# 用途：解锁基础功能、种植胡萝卜

def farm_hay():
    for y in range(get_world_size()):
        for x in range(get_world_size()):
            if can_harvest():
                harvest()
            # 移动...
```

### 木材（Wood）
```python
# 来源：灌木（1个）、树（5个）
# 早期：种灌木（4秒成熟）
# 后期：种树（7秒成熟，但产量5倍）
# 用途：种植胡萝卜

def farm_wood_early():
    # 种植灌木
    plant(Entities.Bush)
    while not can_harvest():
        pass
    harvest()

def farm_wood_late():
    # 种植树（棋盘格避免减速）
    if (get_pos_x() + get_pos_y()) % 2 == 0:
        plant(Entities.Tree)
```

### 胡萝卜（Carrot）
```python
# 来源：种植胡萝卜
# 成本：1 木材 + 5 干草
# 用途：种植南瓜

def farm_carrots():
    if get_ground_type() != Grounds.Soil:
        till()
    plant(Entities.Carrot)
    while not can_harvest():
        pass
    harvest()
```

### 南瓜（Pumpkin）
```python
# 来源：种植南瓜（合并机制）
# 成本：1 胡萝卜
# 策略：种植 6×6 或更大，等待合并
# 用途：种植向日葵、仙人掌

def farm_pumpkins():
    size = 6
    # 种植 6×6 网格
    # 等待全部成熟
    # 收割任意一个（获得 216 个南瓜）
```

### 能量（Power）
```python
# 来源：向日葵
# 成本：25 南瓜
# 策略：种植 10+ 株，收割最大花瓣（5× 奖励）
# 效果：所有操作 2× 速度

def farm_power():
    # 种植至少 10 株向日葵
    # 找到最大花瓣的那株
    # 收割获得 5× 能量
```

### 仙人掌（Cactus Item）
```python
# 来源：仙人掌（排序机制）
# 成本：5 南瓜
# 策略：排序后收割（n 个 = n² 产量）
# 用途：恐龙帽（购买苹果）

def farm_cactus():
    # 种植仙人掌网格
    # 排序仙人掌
    # 收割获得 n² 产量
```

### 奇异物质（Weird_Substance）
```python
# 来源1：施肥的植物（收获时 50% 产量转化）
# 来源2：收获感染的植物
# 用途：生成迷宫、切换感染状态

def get_weird_substance():
    # 种植作物
    # 使用肥料（感染）
    # 收获（一半产量变奇异物质）
```

### 金币（Gold）
```python
# 来源：迷宫宝藏
# 策略：生成迷宫，导航到宝藏，收获
# 产量：迷宫面积（n×n = n² 金币）

def farm_gold():
    plant(Entities.Bush)
    substance = get_world_size() * (2 ** (num_unlocked(Unlocks.Mazes) - 1))
    use_item(Items.Weird_Substance, substance)
    # 导航到宝藏
    # harvest()
```

### 骨头（Bone）
```python
# 来源：恐龙帽
# 策略：装备恐龙帽，覆盖农场，卸下帽子
# 产量：尾巴长度²

def farm_bones():
    change_hat(Hats.Dinosaur_Hat)
    # 遍历整个农场（哈密尔顿路径）
    change_hat(Hats.None)  # 获得 (length²) 骨头
```

## 常用常量

### 方向
```python
North  # 向上（y+1）
South  # 向下（y-1）
East   # 向右（x+1）
West   # 向左（x-1）

# 相反方向
def opposite(direction):
    if direction == North:
        return South
    if direction == South:
        return North
    if direction == East:
        return West
    if direction == West:
        return East
```

### 实体类型
```python
# 检查实体
Entities.Grass
Entities.Bush
Entities.Tree
Entities.Carrot

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nql1314/The-Farmer-Was-Replaced-AI-Code](https://github.com/nql1314/The-Farmer-Was-Replaced-AI-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
