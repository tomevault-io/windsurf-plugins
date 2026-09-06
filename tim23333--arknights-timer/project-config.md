---
trigger: always_on
description: cd ark_parser/character
---

# Arknights 干员数据解析工具

## 使用方法

### 批量提取我方全部数据（干员/技能/装置/装备/模组）

```bash
cd ark_parser/character
python extract_character_data.py
```

输出（`ark_parser/character/data/`）：
- `characters.json` — character_table 全条目（1368 条，含干员/token/trap）
- `skills.json` — 技能完整数据（1803 个，含 skchr_/sktok_）
- `devices.json` — token/trap/装置分类
- `battle_equip.json` / `uniequip.json` — 战斗装备 / 模组

### 批量提取敌方数据

```bash
cd ark_parser/enemy
python extract_enemy_data.py
```

输出（`ark_parser/enemy/data/`）：`enemy_database.json`、`enemy_handbook.json`、
`stage_enemy_usage.json` 等。游戏更新后配套运行
`python -m tools.enemy_health.update_from_unpack --assets <热更解包目录>`
重建内存偏移（generated_offsets.json）与敌人名称库（enemy_names.json）。

**数值显示规则：** 所有从 blackboard 提取的数值必须转换为标准十进制显示，不要显示原始的 IEEE 754 浮点二进制整数（如 `1065353216` → `1.0`）。转换函数：
```python
import struct
def i2f(val):
    if isinstance(val, int) and 0 <= val <= 4294967295:
        return round(struct.unpack('<f', struct.pack('<I', val))[0], 4)
    return val
```

### 提取我方/敌方动作生效帧

```bash
python ark_parser/extract_effect_frames.py    # 需 UnityPy（已含在 backend/requirements.txt，.venv 可直接跑）
```

输出：`data/tables/effect_frames.json` — 每个敌人/干员的普攻与技能：
动画名、Spine 动画事件生效帧（`OnAttack` 等，秒+帧）、弹道速度（或固定飞行时间）。
数据源：`data/refs/arts/enm_art_*`（敌人 spine）、`data/chararts`（我方 spine）、
`data/battle/enm_pfb_*`/`data/charpack`（Ability 参数 `_animKey/_preDelay/_projectileKey`）、
`data/battle/prefabs/[uc]projectiles.ab_unpacked`（弹道移动组件）。
运行时由 `tools/enemy_health/enemy_db.load_effect_frames()` 加载，在敌我详情页
「生效帧」tab 展示。

### 快速查看干员数据

```bash
# 查看干员基础信息
python -c "import json; d=json.load(open('ark_parser/character/data/characters.json',encoding='utf-8')); print(json.dumps(d.get('char_1045_svash2',{}), indent=2, ensure_ascii=False))"

# 查看技能描述（skills.json: skillId -> {levels: [{name, description, blackboard, ...}]})
python -c "import json; d=json.load(open('ark_parser/character/data/skills.json',encoding='utf-8')); [print(k, v['levels'][0].get('name',''), v['levels'][0].get('description','')[:80]) for k,v in d.items() if 'svash2' in k]"
```

### 查看干员技能详细文本

在 `skills.json` 中搜索技能 ID（格式：`skchr_<代号>_<1/2/3>`）即可查看技能描述和 blackboard 参数。

---

## 干员 ID 命名规则

格式：`char_<编号>_<代号>`

| 干员 | ID |
|---|---|
| 阿米娅 | char_002_amiya |
| 凯尔希 | char_003_kalts |
| 陈 | char_010_chen |
| 凛御银灰 | char_1045_svash2 |
| 斯卡蒂 | char_263_skadi |

代号通常为英文缩写，异格干员编号 1000+。

---

## 数据来源文件

### 自动提取流程

游戏数据分两层：**基础包**（`StreamingAssets/AB/Windows/anon/`，随大版本安装/重装更新）
和**热更包**（`Arknights_Data/PersistentData/Bundles/anon/`，每次热更新下载，覆盖同名表）。
提取时必须 base+hot 合并，hot 覆盖 base。

1. 用 ArknightsStudioCLI 以 **exportRaw** 模式解出剥离版 TextAsset（base 与 hot 各跑一遍）：
```bash
CLI="AssetStudio-ArknightsStudio/AssetStudioCLI/bin/Release/net8.0/ArknightsStudioCLI.exe"
"$CLI" "<游戏>\Arknights_Data\StreamingAssets\AB\Windows\anon"        -t textAsset -m exportRaw -g none -o <base_out>
"$CLI" "<游戏>\Arknights_Data\PersistentData\Bundles\anon"            -t textAsset -m exportRaw -g none -o <hot_out>
```
2. 把两批 `.dat` 表文件按 `data/anon/<名字>.bin_unpacked/CAB-*` 布局摆放
   （目录名排序后者优先：base 用 `base_*`，热更用 `zz_hot_*`），然后运行：
```bash
python extract_tables.py
```

脚本扫描 `data/anon/*.bin_unpacked/CAB-*`（按目录名排序，**后扫到的覆盖先扫到的**，
保证热更表生效），识别表名并提取到 `data/tables/`。

> **格式陷阱：** AssetStudio GUI「Extract folder」产出的 `CAB-*` 是 Unity
> SerializedFile（全零头），**不能**直接被 `FB` 解析器读取；只有 exportRaw
> 剥离版（`[u32 名字长度][表名][128B 签名头][FlatBuffers]`）可以。2026-08 起
> data/anon 只放剥离版伪解包目录。

当前基线（2026-08-11）：base = 2026-07-22 安装版 + hot = 2026-08-11 热更，
快照在 `unpack_work/all_tables_20260801_base/` 与 `unpack_work/hot_20260811/raw/`。

### 提取的数据表

| 表名 | 内容 |
|---|---|
| character_table | 干员基础数据 |
| skill_table | 技能完整数据 |
| stage_table | 关卡数据 |
| activity_table | 活动数据 |
| charword_table | 干员语音/台词 |
| handbook_info_table | 干员档案 |
| uniequip_table | 模组数据 |
| battle_equip_table | 战斗装备 |
| skin_table | 皮肤数据 |
| retro_table | 复刻活动数据 |
| roguelike_topic_table | 肉鸽主题数据 |
| sandbox_perm_table | 沙盒权限数据 |
| building_data | 基建数据 |
| enemy_handbook_table | 敌人图鉴 |
| enemy_database | 敌人战斗数据 |
| item_table / gacha_table / medal_table | 物品/抽卡/蚀刻章 |
| story_table / zone_table | 剧情/区域 |
| shop_client_table / climb_tower_table | 商店/爬塔 |
| arkvent_table / battle_misc_table | 活动副本/战斗杂项 |
| display_meta_table / extra_battlelog_table / hotupdate_meta_table | 展示元数据/额外战报/热更元数据 |

### 原始 AB 文件位置

基础包：`E:\Hypergryph Launcher\games\Arknights Game\Arknights_Data\StreamingAssets\AB\Windows\anon\`
热更包：`E:\Hypergryph Launcher\games\Arknights Game\Arknights_Data\PersistentData\Bundles\anon\`

> **注意：** AB 包文件名的 hash 后缀会随游戏版本更新变化，`extract_tables.py` 通过表名前缀匹配，无需手动更新文件名。

---

## 推理过程记录

### 1. 发现游戏数据存储位置

- 游戏使用 Unity 2021.3.39f1 引擎，IL2CPP 编译
- 数据表存储在 `StreamingAssets/AB/Windows/anon/` 目录的加密 AB 包中
- AB 包使用 LZHAM 压缩（Unity 已弃用的压缩格式）
- 使用 AssetStudio-Arknights 分支（支持 Arknights 自定义格式）解包 AB 文件
- 解包后得到 TextAsset 类型的二进制文件

### 2. 破解二进制格式

**文件结构：**
```
Offset 0-127:    加密/混淆头部（128字节）
Offset 128-131:  版本号（uint32）
Offset 136-139:  计数（uint32）
Offset 140-143:  条目数（uint32）= 1127（干员表）/ 1604（技能表）
Offset 144+:     索引数据
Offset ~311K+:   数据区域（Region 0 和 Region 1）
Offset ~2.3M:    字符串表（中文文本、描述）
```

**格式识别：**
- 数据是 FlatBuffers 序列化格式（不是标准 FlatBuffers，是 Arknights 自定义变体）
- 通过分析 `dump.cs` 中的 `Table` 类确认（有 `__offset`、`__string`、`__vector` 方法）
- `FlatLookupConverter` 类的 `Unpack_*` 方法处理反序列化

### 3. FlatBuffers 解析核心

**类型检测逻辑（在 `parse_value` 函数中）：**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tim23333/Arknights_timer](https://github.com/Tim23333/Arknights_timer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
