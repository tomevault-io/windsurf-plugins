---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

EmbeddedGUI 是一个轻量级的 C 语言 GUI 框架，专为资源受限的嵌入式系统设计（RAM <8KB，ROM <64KB，CPU ~100MHz，无需浮点运算单元）。采用局部帧缓冲（PFB）设计，在最小化内存使用的同时支持触控、动画和类 Android 的 UI 模式。

## 构建命令

```bash
# 构建并运行（PC 模拟器）
make all APP=HelloSimple
make run

# 指定平台构建
make all APP=HelloActivity PORT=pc
make all APP=HelloSimple PORT=stm32g0

# 构建 HelloBasic 并选择子应用
make all APP=HelloBasic APP_SUB=button
make all APP=HelloBasic APP_SUB=anim

# 清理构建产物
make clean

# 生成资源文件
make resource

# 分析二进制大小（生成 output/README.md）
python scripts/size_analysis/main.py

# 格式化代码
python scripts/code_format.py

# 完整编译检查（CI 使用）
python scripts/code_compile_check.py --full-check

# 使用 CMake 构建
cmake -B build_cmake/HelloSimple -DAPP=HelloSimple -DPORT=pc -G "MinGW Makefiles"
cmake --build build_cmake/HelloSimple -j

# CMake 构建 HelloBasic 子应用
cmake -B build_cmake/HelloBasic_button -DAPP=HelloBasic -DAPP_SUB=button -DPORT=pc -G "MinGW Makefiles"
cmake --build build_cmake/HelloBasic_button -j

# 使用 CMake 做完整编译检查
python scripts/code_compile_check.py --cmake --full-check
```

**可用示例：** HelloActivity, HelloAPP, HelloBasic, HelloCanvas, HelloChart, HelloEasyPage, HelloGradient, HelloLayer, HelloMultiDisplay, HelloMultiDisplayHetero, HelloPerformance, HelloPFB, HelloResourceManager, HelloShowcase, HelloSimple, HelloSizeAnalysis, HelloStyleDemo, HelloTest, HelloUnitTest, HelloViewPageAndScroll, HelloVirtual

**HelloBasic 子应用：** activity_ring, analog_clock, anim, animated_image, arc_slider, button, button_img, button_matrix, card, checkbox, circular_progress_bar, combobox, compass, digital_clock, divider, enhanced_widgets, gauge, gridlayout, heart_rate, image, image_button, label, led, line, linearlayout, list, mask, menu, mini_calendar, mp4, notification_badge, number_picker, page_indicator, progress_bar, radio_button, roller, rotation, scale, scroll, slider, spangroup, spinner, stopwatch, svg, switch, table, tab_bar, textblock, textinput, tileview, toggle_button, viewpage, viewpage_cache, window

## 架构说明

### 目录结构
- `src/` - 核心库（控件、动画、字体、图片、遮罩）
- `example/` - 示例应用
- `porting/` - 平台移植（pc, stm32g0, qemu, emscripten, designer, pc_test）
- `scripts/` - 构建自动化和资源生成工具

### 面向对象的 C 语言模式
使用基于结构体的"类"和函数指针实现面向对象。基类为 `egui_view_t`，通过组合实现继承：
```c
static egui_view_button_t button;
egui_view_button_init(&button);
egui_view_set_on_click_listener(&button, callback);
```

### OOP 辅助宏（egui_oop.h）

项目提供了一套 OOP 辅助宏，用于简化类型转换，零运行时开销，兼容 C99：

| 宏 | 用途 | 示例 |
|----|------|------|
| `EGUI_LOCAL_INIT(_type)` | self 指针向下转换 | `EGUI_LOCAL_INIT(egui_view_label_t);` |
| `EGUI_VIEW_OF(_ptr)` | 向上转换为 egui_view_t* | `EGUI_VIEW_OF(&local->container)` |
| `EGUI_ANIM_OF(_ptr)` | 向上转换为 egui_animation_t* | `EGUI_ANIM_OF(&local->anim)` |
| `EGUI_VIEW_PARENT(_view)` | 获取父视图指针 | `EGUI_VIEW_PARENT(self)` |
| `EGUI_CAST_TO(_type, _ptr)` | 显式向下转换 | `EGUI_CAST_TO(egui_view_label_t, ptr)` |

**使用示例**：
```c
// 改造前
void egui_view_label_set_text(egui_view_t *self, const char *text)
{
    egui_view_label_t *local = (egui_view_label_t *)self;
    local->text = text;
}

// 改造后
void egui_view_label_set_text(egui_view_t *self, const char *text)
{
    EGUI_LOCAL_INIT(egui_view_label_t);
    local->text = text;
}
```

### 核心源码模块
- `src/core/` - 事件循环、画布、定时器、输入处理、配置
- `src/widget/` - UI 控件（view, button, label, scroll, viewpage 等）
- `src/app/` - Activity、Dialog、Toast（类 Android 生命周期）
- `src/anim/` - 动画和 9 种插值器
- `src/utils/` - 定点数运算、链表、环形缓冲区

### 配置系统
每个示例可通过 `app_egui_config.h` 覆盖默认配置：
```c
#define EGUI_CONFIG_SCREEN_WIDTH 240
#define EGUI_CONFIG_SCREEN_HEIGHT 320
#define EGUI_CONFIG_COLOR_DEPTH 16
#define EGUI_CONFIG_PFB_WIDTH (240/8)
#define EGUI_CONFIG_PFB_HEIGHT (320/8)
```

### 构建模块系统
每个组件都有一个 `build.mk` 文件，定义 `EGUI_CODE_SRC` 和 `EGUI_CODE_INCLUDE`。根目录的 Makefile 包含来自 src/、example/ 和 porting/ 的模块。

## 核心设计理念

1. **局部帧缓冲（PFB）**：不使用全屏帧缓冲，而是使用小块缓冲（如 30x40 像素 = RGB565 下 2400 字节）。宽度/高度必须是屏幕尺寸的整数约数。
2. **脏矩形机制**：仅重绘变化的区域，降低功耗。
3. **定点数运算**：所有计算使用定点数（无浮点），实现在 `egui_fixmath.h`。
4. **类 Android API**：Activity 管理页面，ViewPage 实现滑动切页，LinearLayout 进行布局。

## 交互说明

- 交互都用简体中文

- 代码用utf-8英文，代码里面不要出现中文。

- 文档使用utf-8简体中文

## 代码风格

使用基于 LLVM 的代码风格，自定义配置见 `.clang-format`：
- 4 空格缩进，不使用 Tab
- 160 列宽限制
- 函数和控制语句的大括号换行

## 运行与渲染验证

每次修改代码并构建成功后，必须执行运行时验证，确保程序不会卡死、崩溃，且渲染结果正确：

```bash
# 对具体示例进行运行验证
python scripts/code_runtime_check.py --app HelloBasic --app-sub textinput --timeout 10

# 验证所有示例
python scripts/code_runtime_check.py --full-check
```

验证要点：
- 构建成功后必须运行 `code_runtime_check.py` 进行运行时验证
- 检查截图输出（`runtime_check_output/` 目录下的 PNG 文件）确认渲染正确
- 检查文本、图标、中心按钮等关键元素的视觉居中是否准确，左右/上下留白是否平衡；不能只以“没截断”作为通过标准
- 检查文字与按钮/圆形/胶囊等边框之间是否保留合理空隙，不能出现文字贴边、视觉压迫或左右内边距明显失衡
- HelloCustomWidgets 已迁移到独立仓库 `EmbeddedGUI_Widgets`；如任务涉及 custom widgets、其 iteration_log 或对应 workflow，请切换到该仓库执行
- iteration_log/ 属于本地审阅产物，不纳入 git 提交
- 如果运行时检查失败（卡死、崩溃），必须排查修复后重新验证
- **多页面应用必须验证所有页面渲染**：对于包含多个页面的应用，运行时验证必须覆盖每一个页面，不能只验证首页。代码生成器会自动为多页面项目生成 `egui_port_get_recording_action()` 录制动作，通过 `uicode_switch_page()` 依次切换所有页面。验证时需检查截图确认每个页面都有正确渲染输出
- **性能测试必须使用 QEMU 验证**：PC 模拟器的计时器精度只有 1ms，不适合做性能基准测试。性能数据必须通过 QEMU 运行获取（使用微秒级计时器 `qemu_get_tick_us`）。PC 运行仅用于方便查看渲染效果，不作为性能数据依据

## UI Designer 迁移说明


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EmbeddedGUI/EmbeddedGUI](https://github.com/EmbeddedGUI/EmbeddedGUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
