---
trigger: always_on
description: 本文档是 Alas（AzurLaneAutoScript）项目的开发规范，讲述框架和设计模式，介绍常用组件，帮助新开发者熟悉项目。具体函数的使用说明应参阅 API 文档或源码。
---


# Alas 开发文档规则

本文档是 Alas（AzurLaneAutoScript）项目的开发规范，讲述框架和设计模式，介绍常用组件，帮助新开发者熟悉项目。具体函数的使用说明应参阅 API 文档或源码。

---

## 1. 基本运作模式与设计原则

### 1.1 Alas 的应用场景

- Alas 是为长时间运行（7×24 小时）而设计的
- 放弃安卓真机支持的原因：
  - 安卓机在长时间运行下容易出现黑屏/假死
  - 部分安卓机型截图会压缩
  - OCR 模型迁移困难
- 仅支持 1280×720 分辨率：
  - 720p 在图像清晰度和截图耗时之间有较好平衡
  - 异型屏没有统一标准，适配成本高

### 1.2 状态循环模式（核心原则）

**禁止使用"点击-等待"模式：**

```python
# 禁止这样写
click(XXXX)
sleep(2)
click(YYYY)
sleep(3)
```

**必须使用"状态循环"模式：**

```python
while 1:
    self.device.screenshot()

    if self.appear_then_click(ENTRANCE):
        continue
    if self.appear_then_click(MAP_PREPARATION):
        continue
    if self.appear_then_click(FLEET_PREPARATION):
        continue

    # End
    if self.handle_in_map_with_enemy_searching():
        break
```

- 状态循环在高配电脑上运行快，在低配电脑上也有很好的兼容性
- 可以在点击失败时自动重试
- 不需要关心点击的执行顺序

### 1.3 处理死循环

Alas 内置两种异常来检测死循环：

- **GameStuckError**: 无操作连续截图超过 1 分钟（战斗中和客户端启动中延长至 5 分钟）
- **GameTooManyClickError**: 最后 15 次操作中，有一项操作 ≥12 次，或有两项操作都 ≥6 次

### 1.4 性能优化

- 开发者在编写 Alas 时不需要特别注意性能优化
- Alas 运行超过 99% 的时间是在等待模拟器截图
- 截图耗时约 350ms，处理只花费约 2.5ms
- 海图识别或 OCR 耗时约 100-180ms

---

## 2. 注释规范

### 2.1 Google 注释规范

使用 Google 注释规范，例如：

```python
"""
Re-focus to the center of a grid.

Args:
    tolerance (float): 0 to 0.5. If None, use MAP_GRID_CENTER_TOLERANCE

Returns:
    bool: Map swiped.
"""
```

### 2.2 Pages 注释

增加 Pages 说明函数进出时的游戏界面：

```python
"""
Pages:
    in: page_moewfficer
    out: MEOWFFICER_BUY
"""
```

### 2.3 注释要求

- 在注释中，应当全部使用简体中文

**尽量做到：**

- 一个函数的注释占 1/3 ~ 1/2
- 一个函数不超过一个屏幕
- 一个 .py 文件不超过 500 行

---

## 3. 调试规范

### 3.1 调试入口

Alas 的入口文件有两个：
- 调度器 `alas.py`
- 网页后端 `gui.py`

### 3.2 调试一个 Button

```python
# 假设 Alas 无法识别 SOS 模块中的 SIGNAL_LIST_CHECK
# 将游戏切换到 SOS 信号列表的界面，截图

az = CampaignSos('alas', task='Sos')
az.image_file = r'xxxxx.png'

print(az.appear(SIGNAL_LIST_CHECK))
```

### 3.3 调试其他服务器

在导入任何 Alas 内容之前，切换服务器：

```python
import module.config.server as server
server.server = 'en'
```

### 3.4 调试一个识别函数

```python
from module.statistics.utils import load_folder
from module.handler.info_handler import InfoHandler

folder = r'xxxxx'
az = InfoHandler('alas', task='Alas')
for file in load_folder(folder).values():
    az.image_file = file
    print(az._story_option_buttons())
```

### 3.5 调试海图识别

```python
from PIL import Image
from module.config.config import AzurLaneConfig
from module.map_detection.view import *

file = r'xxxxx'

class Config:  # 把地图文件中的 Config 粘贴到这里
    pass

md = View(AzurLaneConfig('template').merge(Config()))
image = np.array(Image.open(file).convert('RGB'))
md.load(image)
md.predict()
md.show()
md.backend.draw()
```

### 3.6 为测试提供方便

- Alas 所有模块都可以独立运行，不依赖 GUI 也不依赖用户配置
- 每个模块通常只有一个方法是依赖用户配置的
- 提供两份方法：例如 `fleet_repair` 和 `handle_port_repair`

---

## 4. 工具类（module/base/utils.py）

### 4.1 命名约定

- **point**: 含 2 个元素的 tuple (x, y)，指屏幕上的一个点
- **area**: 含 4 个元素的 tuple (upper_left_x, upper_left_y, bottom_right_x, bottom_right_y)
- **location**: 含 2 个元素的 tuple (x, y)，指游戏海域中的网格坐标
- **node**: str 类型，如 "E3"，指游戏海域中的网格坐标

### 4.2 常用工具函数

- `random_normal_distribution_int(a, b, n=3)`: 在区间 [a, b) 内产生符合正态分布的随机数
- `random_rectangle_point(area)`: 在区域内产生符合二维正态分布的随机点
- `random_rectangle_vector(vector, box, random_range, padding)`: 在区域按二维正态分布放置向量
- `random_line_segments(p1, p2, n, random_range)`: 在两点之间插入中间值
- `crop(image, area)`: 裁切图片
- `get_color(image, area)`: 计算区域的平均颜色
- `color_similarity(color1, color2)`: 计算两个颜色之间的差值
- `color_similar(color1, color2, threshold)`: 判断颜色是否相似
- `color_similarity_2d(image, color)`: 计算二维数组上的颜色差值
- `extract_letters(image, letter, threshold)`: 将含文字的图片转换为白底黑字
- `color_bar_percentage(image, area, prev_color, reverse, starter, threshold)`: 计算进度条的百分比

---

## 5. 装饰器（module/base/decorators.py）

### 5.1 @Config.when()

让一个函数在特定的设置情况下运行：

```python
from module.base.decorator import Config
from module.base.base import ModuleBase

class AnotherModule(ModuleBase):
    @Config.when(SERVER='en')
    def function(self):
        # 此方法将仅在 EN 服务器中调用
        pass

    @Config.when(SERVER=None)
    def function(self):
        # 此方法将在其他服务器中调用
        pass
```

### 5.2 @cached_property

缓存属性，只计算一次：

```python
@cached_property
def bug_threshold(self):
    return random_normal_distribution_int(55, 105, n=2)
```

### 5.3 @timer

打印函数运行的耗时：

```python
@timer
def do_something():
    pass
```

### 5.4 @function_drop(rate, default)

随机执行或者不执行某个函数：

```python
@function_drop(rate=0.5, default=None)
def some_function():
    pass
```

---

## 6. 日志规范（module/logger.py）

### 6.1 日志格式

- 格式: `%(asctime)s.%(msecs)03d | %(levelname)s | %(message)s`
- 时间格式: `%Y-%m-%d %H:%M:%S`
- 示例: `2020-09-11 08:35:59.460 | INFO | XXXXXXXX`

### 6.2 日志级别使用规范

- **logger.hr(title, level=0)**: 仅在脚本开始运行时使用
- **logger.hr(title, level=1)**: 表示开始执行 GUI 中的某个功能
- **logger.hr(title, level=2)**: 表示功能的某个阶段的开始
- **logger.hr(title, level=3)**: 表示功能的某个细分阶段的开始
- **logger.attr(name, text)**: 用于打印属性值
- **logger.attr_align(name, text, front, align)**: 用于打印属性值，有一定格式

### 6.3 日志注意事项

- log 需要人工阅读，请避免大量使用 `logger.hr()`、`logger.warning()`、感叹号等表示强调
- 强调是相对的，如果强调了所有事物，相当于没有强调任何一个事物

---

## 7. 异常处理（module/exception.py）

### 7.1 主要异常类

- **CampaignEnd**: 关卡战斗结束
- **MapDetectionError**: 海域地图识别错误
- **MapWalkError**: 无法移动至目标点
- **MapEnemyMoved**: 敌人已经移动，需要重新进行地图扫描

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wess09/AzurPilot](https://github.com/wess09/AzurPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
