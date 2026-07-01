---
trigger: always_on
description: - 新无人机通过 `spawn_drone(function)` 生成
---

# TFWR 游戏机制详解

## 巨型农场（Megafarm）- 多无人机系统

### 基础概念
- 允许使用多架无人机同时工作
- 每架无人机运行独立的程序
- 新无人机通过 `spawn_drone(function)` 生成
- 无人机之间不会碰撞

### 无人机管理函数
```python
# 生成新无人机
drone = spawn_drone(drone_function)

# 获取无人机数量上限
max_count = max_drones()

# 获取当前无人机数量
current_count = num_drones()

# 等待无人机完成
result = wait_for(drone)

# 检查无人机是否完成（不等待）
if has_finished(drone):
    pass
```

### 生成无人机示例
```python
# 示例1：简单的列收割
def harvest_column():
    for _ in range(get_world_size()):
        harvest()
        move(North)

while True:
    if spawn_drone(harvest_column):
        move(East)
    else:
        break  # 无法生成更多无人机

# 示例2：传递不同参数
for dir in [North, East, South, West]:
    def task():
        move(dir)
        do_a_flip()
    spawn_drone(task)
```

### 实用模式：并行 for_all
```python
# 在整个农场上并行执行函数
def for_all(f):
    def row():
        for _ in range(get_world_size()-1):
            f()
            move(East)
        f()
    
    for _ in range(get_world_size()):
        if not spawn_drone(row):
            row()  # 如果无法生成无人机，自己执行
        move(North)

# 使用示例
for_all(harvest)
```

### 条件生成模式
```python
# 如果有可用无人机就生成，否则自己执行
if not spawn_drone(task):
    task()
```

### 重要限制

#### 1. 内存隔离与共享（已验证）

**基本规则：通过闭包捕获的变量会被复制，无人机间不共享**

```python
# ❌ 错误：无人机不共享全局变量
x = 0

def increment():
    global x
    x += 1

wait_for(spawn_drone(increment))
print(x)  # 仍然是 0！无人机修改了自己的副本

# ❌ 错误：列表也不共享（会被复制）
shared_list = [0, 0, 0]

def modify_list():
    shared_list[0] = 100  # 只修改副本，不影响原列表

drone = spawn_drone(modify_list)
wait_for(drone)
print(shared_list)  # 仍然是 [0, 0, 0]！
```

**高级技巧：通过 wait_for() 实现共享内存（重大发现！）**

多个无人机可以通过 `wait_for()` 同一个源无人机来获取**共享的引用类型对象**！

```python
# ✅ 共享内存模式：创建共享数据源
def create_shared():
    return []  # 返回一个列表

source = spawn_drone(create_shared)

# 多个工作者通过 wait_for 获取同一个列表引用
def worker():
    data = wait_for(source)  # 所有 worker 获得同一个列表！
    data.append(num_drones())  # 修改会相互可见
    print(num_drones(), data)

# 启动多个工作者
for i in range(5):
    spawn_drone(worker)
    do_a_flip()

# 输出示例：
# 2 [2]
# 3 [2,3]
# 4 [2,3,4]
# 5 [2,3,4,5]
# 6 [2,3,4,5,6]
# 证明：所有 worker 共享同一个列表！
```

**共享字典示例（更实用）**

```python
# 创建共享统计字典
def create_stats():
    return {
        "grass": 0,
        "trees": 0,
        "total": 0
    }

stats_source = spawn_drone(create_stats)

# 工作者更新共享统计
def count_zone():
    local_grass = 0
    local_trees = 0
    
    # 扫描区域
    for i in range(25):
        entity = get_entity_type()
        if entity == Entities.Grass:
            local_grass += 1
        elif entity == Entities.Tree:
            local_trees += 1
        move(East)
    
    # 更新共享统计
    stats = wait_for(stats_source)
    stats["grass"] += local_grass
    stats["trees"] += local_trees
    stats["total"] += 25
    
    return local_grass + local_trees

# 启动多个扫描器
drones = []
for i in range(4):
    drone = spawn_drone(count_zone)
    if drone:
        drones.append(drone)

# 等待完成
for drone in drones:
    wait_for(drone)

# 获取最终统计
final_stats = wait_for(stats_source)
quick_print("Total grass:", final_stats["grass"])
quick_print("Total trees:", final_stats["trees"])
```

**⚠️ 重要警告：竞态条件风险**

```python
# ❌ 危险：竞态条件
def unsafe():
    data = wait_for(source)
    count = data["count"]  # 读取
    count += 1             # 计算
    data["count"] = count  # 写入 - 可能覆盖其他无人机的修改

# ✅ 较安全：原子操作
def safer():
    data = wait_for(source)
    data["count"] += 1  # 单步操作

# ✅ 最安全：只追加
def safest():
    data = wait_for(source)
    data.append(result)  # 追加操作

# ✅ 最安全：使用独立键
def best():
    data = wait_for(source)
    drone_id = num_drones()
    data[drone_id] = result  # 每个无人机有自己的键
```

**使用建议**

- ✅ **优先使用返回值通信**：大多数情况下更安全简单
- ⚠️ **谨慎使用共享内存**：只在需要实时协作时使用
- ✅ **只追加不修改**：使用 `append()` 而不是修改现有值
- ✅ **使用独立键**：每个无人机操作不同的字典键
- ❌ **避免读-修改-写**：容易产生竞态条件

#### 2. 竞态条件
```python
# ❌ 危险：多个无人机可能同时执行
if get_water() < 0.5:
    use_item(Items.Water)  # 可能多个无人机都会浇水

# ✅ 正确：避免多个无人机操作同一地块
# 将农场分区，每个无人机负责不同区域
```

#### 3. 生成成本
- 生成无人机需要时间（约200 ticks）
- 不要为每个小任务都生成无人机
- 适合长时间运行的任务

## 迷宫（Mazes）

### 生成迷宫
```python
# 在灌木上使用奇异物质生成迷宫
plant(Entities.Bush)

# 计算所需的奇异物质数量
# 基础：n 份奇异物质 = n×n 迷宫
# 每次升级：需要 2 倍物质，宝藏金币也翻倍
maze_upgrades = num_unlocked(Unlocks.Mazes) - 1
substance_needed = get_world_size() * (2 ** maze_upgrades)

use_item(Items.Weird_Substance, substance_needed)
```

### 迷宫特性
- 无人机无法飞过树篱（`Entities.Hedge`）
- 宝藏位置：`get_entity_type() == Entities.Treasure`
- 宝藏金币数 = 迷宫面积（如 5×5 = 25 金币）
- 首次生成的迷宫无循环

### 导航迷宫
```python
# 检查是否有墙
if can_move(North):
    move(North)

# 或者尝试移动并检查结果
if move(North):
    # 移动成功
    pass
else:
    # 遇到墙

# 获取宝藏位置
treasure_x, treasure_y = measure()
```

### 收获宝藏
```python
# 在宝藏上收获获得金币
if get_entity_type() == Entities.Treasure:
    harvest()  # 获得金币，迷宫消失
```

### 重复使用迷宫（高级）
```python
# 在宝藏上再次使用奇异物质
# 宝藏会移动到随机位置，金币增加
use_item(Items.Weird_Substance, substance_needed)

# 注意：
# - 重复使用会移除一些墙，可能产生循环
# - 最多重复 300 次
# - 金币不会比新建迷宫更多
# - 仅作为额外挑战
```

### 迷宫算法提示
```python
# 推荐使用深度优先搜索（DFS）或广度优先搜索（BFS）
# 需要记录已访问位置，避免死循环

# DFS 示例框架
visited = set()

def explore(x, y):
    if (x, y) in visited:
        return
    
    visited.add((x, y))
    
    if get_entity_type() == Entities.Treasure:
        harvest()
        return
    
    # 尝试四个方向
    for direction in [North, East, South, West]:
        if can_move(direction):
            move(direction)
            new_x, new_y = get_pos_x(), get_pos_y()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nql1314/The-Farmer-Was-Replaced-AI-Code](https://github.com/nql1314/The-Farmer-Was-Replaced-AI-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
