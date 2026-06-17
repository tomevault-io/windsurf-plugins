---
trigger: always_on
description: FC 语言完整语法和 API 参考
---


# FC 语言 Import 映射表

使用原则：每次使用函数前，查询此表补充 import 语句

---

## 核心规则

### 必须 Import 的情况
1. 非 StdLibrary 的函数 - 必须查表并添加 import
2. 非 StdLibrary 的事件 - 必须查表并添加 import（重要！）
3. List、Map 操作 - 必须 import 对应库
4. 数学计算函数 - 必须 import "Math.fcc"
5. 物理检测函数 - 必须 import "Physics.fcc"
6. 战斗相关函数 - 必须 import "Combat.fcc"

### 无需 Import 的情况
1. StdLibrary 内置函数 - 如 LogInfo(), WaitForMillisecond(), GetAllPlayers()
2. StdLibrary 内置事件 - 如 OnAwake(), OnGameStart(), OnUpdate(), OnDestroy()
3. 语言关键字和基本类型 - 如 Vector3, Quaternion, bool, int
4. 编辑器生成的枚举/资源 - 使用 import "EditorGenLib.fcc"

### 事件 Import 示例
```fc
// 错误：缺少 import
graph ButtonHandler {
    event OnTapped(player entity<Player>) {  // 错误：OnTapped 需要 import "Hud.fcc"
        LogInfo("Button tapped")
    }
}

// 正确：添加 import
import "StdLibrary.fcc" as std
import "Hud.fcc" as hud                      // 添加 import

graph ButtonHandler {
    event OnTapped(player entity<Player>) {  // 正确
        LogInfo("Button tapped")
    }
}
```

---

## 函数到库映射表

### List 操作（List.fcc）
```fc
import "List.fcc" as list

New()           → list.New(length, capacity)
Append()        → list.Append(targetList, value)
Length()        → list.Length(targetList)
RemoveAt()      → list.RemoveAt(targetList, index)
Clear()         → list.Clear(targetList)
Reverse()       → list.Reverse(targetList)
Clone()         → list.Clone(targetList)
Insert()        → list.Insert(targetList, index, value)
Shuffle()       → list.Shuffle(targetList)
IsEqual()       → list.IsEqual(listA, listB)
Contain()       → list.Contain(targetList, value)
AppendRange()   → list.AppendRange(targetList, sourceList)
Remove()        → list.Remove(targetList, value)
IndexOf()       → list.IndexOf(targetList, value)
LastIndexOf()   → list.LastIndexOf(targetList, value)
Max()           → list.Max(targetList, out isSuccess, out result)
Min()           → list.Min(targetList, out isSuccess, out result)
```

### Map 操作（Map.fcc）
```fc
import "Map.fcc" as map

New()           → map.New()
Remove()        → map.Remove(targetMap, key)
Clear()         → map.Clear(targetMap)
Length()        → map.Length(targetMap)
ContainKey()    → map.ContainKey(targetMap, key)    // 也可用内置的 ContainKey()
GetAllKeys()    → map.GetAllKeys(targetMap)
```

### 数学运算（Math.fcc）
```fc
import "Math.fcc" as math

// 常量
NaturalConstantType.Pi  → 3.14159265
NaturalConstantType.E   → 2.71828183

// 随机数
RandomInt()             → math.RandomInt(min, max)
RandomFloat()           → math.RandomFloat(min, max)
CreateRandomSeed()      → math.CreateRandomSeed(seed)
Next()                  → math.Next(randomGen, min, max)

// 基础数学
Abs()                   → math.Abs(value)
Sqrt()                  → math.Sqrt(value)
Negate()                → math.Negate(value)
Floor()                 → math.Floor(value)
Ceil()                  → math.Ceil(value)
Round()                 → math.Round(value)
Max()                   → math.Max(a, b)
Min()                   → math.Min(a, b)
Clamp()                 → math.Clamp(value, min, max)
Lerp()                  → math.Lerp(a, b, t)

// 三角函数
Sin()                   → math.Sin(degree)
Cos()                   → math.Cos(degree)
Tan()                   → math.Tan(degree)
ASin()                  → math.ASin(value)
ACos()                  → math.ACos(value)
ATan2()                 → math.ATan2(y, x)

// 幂运算
Exponentiation()        → math.Exponentiation(base, exponent)
Logarithm()             → math.Logarithm(baseValue, realNumber)

// Vector3 运算
Dot()                   → math.Dot(vecA, vecB)
Cross()                 → math.Cross(vecA, vecB)
Normalize()             → math.Normalize(vec)
Magnitude()             → math.Magnitude(vec)
Distance()              → math.Distance(vecA, vecB)
Angle()                 → math.Angle(vecA, vecB)
Projection()            → math.Projection(vecA, vecB)
LerpVector3()           → math.LerpVector3(vecA, vecB, t)
SlerpVector3()          → math.SlerpVector3(vecA, vecB, t)
Vector3Reflect()        → math.Vector3Reflect(direction, normal)

// Vector2 运算
Vector2Dot()            → math.Vector2Dot(vecA, vecB)
Vector2Distance()       → math.Vector2Distance(vecA, vecB)
Vector2Angle()          → math.Vector2Angle(vecA, vecB)
Vector2Normalize()      → math.Vector2Normalize(vec)
Vector2Magnitude()      → math.Vector2Magnitude(vec)
Vector2Reflect()        → math.Vector2Reflect(direction, normal)

// 四元数运算
QuaternionToEulerAngle()    → math.QuaternionToEulerAngle(quat)
EulerAngleToQuaternion()    → math.EulerAngleToQuaternion(euler)
AxisAngleToQuaternion()     → math.AxisAngleToQuaternion(axis, angle)
QuaternionToAxisAngle()     → math.QuaternionToAxisAngle(quat, out axis, out angle)
MultiplyQuaternion()        → math.MultiplyQuaternion(quatA, quatB)
LookRotation()              → math.LookRotation(forward, upward)
SlerpQuaternion()           → math.SlerpQuaternion(quatA, quatB, t)
QuaternionAngle()           → math.QuaternionAngle(quatA, quatB)
QuaternionDot()             → math.QuaternionDot(quatA, quatB)
QuaternionInverse()         → math.QuaternionInverse(quat)
QuaternionFromToRotation()  → math.QuaternionFromToRotation(from, to)
QuaternionRotateTowards()   → math.QuaternionRotateTowards(from, to, maxDegrees)
RotateVector()              → math.RotateVector(vec, deltaQuat)

// 方向转换
DirectionToEulerAngle()     → math.DirectionToEulerAngle(direction)
```

### 字符串操作（Strings.fcc）
```fc

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WindGod-Project-For-Garena/Climbing-World](https://github.com/WindGod-Project-For-Garena/Climbing-World) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
