---
trigger: always_on
description: FC 语言代码示例库
---


# FC 语言开发示例

本文档包含 FC 语言的完整开发示例，每个示例都可以独立使用。

---

## 新手避坑指南

### 示例 1：列表操作

```fc
// 完整正确示例
import "StdLibrary.fcc" as std
import "List.fcc" as list              // 必须 import

graph ListExample {
    MyList List<int>
    
    event OnAwake() {
        MyList = list.New(0, 10)       // 使用命名空间前缀
        list.Append(MyList, 42)
        list.Append(MyList, 100)
        
        var length = list.Length(MyList)
        LogInfo("列表长度: " + length)
        
        for index, value in MyList {
            LogInfo("索引 " + index + " 的值: " + value)
        }
    }
}
```

**常见错误：**
```fc
// 错误：缺少 import "List.fcc"
var myList = New(0, 10)
Append(myList, 42)  // 错误：缺少命名空间前缀
```

---

### 示例 2：组件访问

```fc
// 正确的组件访问
import "StdLibrary.fcc" as std

graph PlayerHandler {
    event OnAwake() {
        // 正确：实例<组件>.属性
        thisEntity<Player>.Health = 100.0
        thisEntity<Player>.RunSpeedScale = 1.5
        thisEntity<Transform>.Position = Vector3{0, 1, 0}
        
        var playerName = thisEntity<Entity>.Name
        LogInfo("玩家名称: " + playerName)
    }
}
```

**常见错误：**
```fc
// 错误 1：使用点号访问组件
thisEntity.Player.Health = 100.0

// 错误 2：缺少组件类型
thisEntity.Health = 100.0

// 错误 3：属性后加尖括号
thisEntity<Player>.Health<float> = 100.0
```

---

### 示例 3：遍历玩家

```fc
import "StdLibrary.fcc" as std

graph PlayerManager {
    event OnAwake() {
        // GetAllPlayers() 是内置函数，无需 import
        for index, player in GetAllPlayers() {
            // player 已经是实例，直接用尖括号访问组件
            player<Player>.RunSpeedScale = 1.5
            player<Player>.Health = 100.0
            player<Transform>.Position = Vector3{index, 1, 0}
            
            var name = player<Entity>.Name
            LogInfo("玩家 " + index + ": " + name)
        }
    }
}
```

**常见错误：**
```fc
for index, player in GetAllPlayers() {
    player.Health = 100.0                     // 错误：使用点号
    player.Transform.Position = Vector3{0, 0, 0}  // 错误
}
```

---

### 示例 4：数学运算

```fc
import "StdLibrary.fcc" as std
import "Math.fcc" as math              // 必须 import

graph MathExample {
    event OnAwake() {
        // 使用命名空间前缀
        var randomNum = math.RandomInt(1, 100)
        var distance = math.Distance(Vector3{0,0,0}, Vector3{10,0,0})
        var normalized = math.Normalize(Vector3{1, 2, 3})
        
        LogInfo("随机数: " + randomNum)
        LogInfo("距离: " + distance)
    }
}
```

**常见错误：**
```fc
var randomNum = RandomInt(1, 100)     // 错误：缺少 import 和命名空间
var distance = Distance(pos1, pos2)   // 错误
```

---

### 示例 5：触发器事件

```fc
import "StdLibrary.fcc" as std

graph TriggerHandler {
    event OnEntityEnter(enterEntity entity<Entity>) {
        // 先检查组件是否存在
        if HasComponent(enterEntity, typeof(Player)) {
            // 转换为 Player 类型
            var player = enterEntity<Player>
            
            // 访问组件
            var playerName = player<Player>.Name
            player<Player>.Health = 150.0
            var position = player<Transform>.Position
            
            LogInfo("玩家进入触发器: " + playerName)
        }
    }
}
```

**常见错误：**
```fc
event OnEntityEnter(enterEntity entity<Entity>) {
    var player = enterEntity<Player>        // 错误：未检查就转换
    var name = enterEntity.Name             // 错误：使用点号
}
```

---

### 示例 6：Map 操作

```fc
import "StdLibrary.fcc" as std
import "Map.fcc" as map                // 必须 import

graph MapExample {
    PlayerScores Map<string, int>
    
    event OnAwake() {
        PlayerScores = map.New()
        
        PlayerScores["Alice"] = 100
        PlayerScores["Bob"] = 85
        
        // ContainKey 是内置函数
        if ContainKey(PlayerScores, "Alice") {
            var score = PlayerScores["Alice"]
            LogInfo("Alice的分数: " + score)
        }
        
        // 获取所有键
        var allKeys = map.GetAllKeys(PlayerScores)
        for index, key in allKeys {
            LogInfo("玩家: " + key + ", 分数: " + PlayerScores[key as string])
        }
    }
}
```

---

### 示例 7：物理检测

```fc
import "StdLibrary.fcc" as std
import "Physics.fcc" as physics        // 必须 import

graph PhysicsExample {
    event OnAwake() {
        var center = Vector3{0, 0, 0}
        var radius = 10.0
        
        for index, player in GetAllPlayers() {
            if physics.IsInsideSphere(player, center, radius) {
                var playerName = player<Player>.Name
                LogInfo(playerName + " 在范围内")
            }
        }
        
        // 射线检测
        physics.SingleRaycast(
            Vector3{0, 1, 0},
            Vector3{0, 0, 1},
            100.0,
            List<int>{},
            false,
            out var hitEntity,
            out var hitPoint,
            out var hitDistance,
            out var hitNormal
        )
        
        if hitEntity != nil {
            LogInfo("射线击中了: " + hitEntity<Entity>.Name)
        }
    }
}
```

---

## 基础示例

### 图形定义

```fc
import "StdLibrary.fcc" as std

// 普通图形
graph MyScript {
    PlayerCount int = 0
    
    event OnAwake() {
        LogInfo("Script initialized")
    }
}

// 静态图形（全局）
static graph GlobalManager {
    MaxPlayers int = 10
}
```

---

### 实体和组件

```fc
import "StdLibrary.fcc" as std

// 访问当前实体组件
graph EntityHandler {
    event OnAwake() {
        thisEntity<Transform>.Position = Vector3{0, 1, 0}
        thisEntity<Transform>.Rotation = Quaternion{0, 0, 0, 1}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WindGod-Project-For-Garena/Climbing-World](https://github.com/WindGod-Project-For-Garena/Climbing-World) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
