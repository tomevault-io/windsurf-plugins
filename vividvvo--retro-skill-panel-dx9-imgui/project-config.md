---
trigger: always_on
description: [文件名] hook_game_text_数据查询索引.txt
---

[文件名] hook_game_text_数据查询索引.txt
[用途] 冒险岛 / Maple 客户端文字接管项目 - 全量查询索引 / 交接总表
[语言] 中文
[状态] 结合本轮对话、上传日志、已确认反编译/断点结果整理；尽量把“已确认 / 已证伪 / 待实现”分开。

============================================================
0. 使用说明 / 查询方法
============================================================
本文件目标：以后需要任何相关数据时，优先在这里全文搜索关键词。

建议搜索关键词：
- 地址：50009C1C / 50009B80 / 5000E640 / 5000E520 / 5000B960 / 50018890 / 004382EA
- 对象：textCtx / drawState / target / FontCache / skillTarget / taskTarget / fullScreenTarget
- 现象：崩溃 / 延迟崩 / 裁剪 / 错位 / 一坨 / 任务框 / 技能列表 / fallback
- 状态：已确认 / 已证伪 / 待实现 / 当前正确方向
- 文件：hook_game_text_takeover_callsite_real_jmp / swaptarget / validated_v2

状态标签说明：
[已确认] 断点、伪代码、现场寄存器/栈或有效 dump 已验证
[已证伪] 已经证明错误，后续不要再走
[候选] 当前看起来可能对，但还没彻底跑通
[历史备份] 有参考价值，但不是当前主路线

============================================================
1. 项目目标与路线历史
============================================================
1.1 当前真正目标 [已确认]
- 在你自己的 D3D9 + ImGui hook 窗口/界面中，借用 Maple 客户端现成的文字链，画出和游戏一致的文字。
- 不是 atlas 最终方案，不是 glyph replay 最终方案，不是 INT3/VEH 最终方案。
- 当前最终目标文件名围绕：hook_game_text_takeover_callsite_real_jmp_swaptarget。

1.2 历史 atlas 路线 [历史备份]
- 旧思路：离线/半离线导出字体 atlas + glyph metrics，再在 ImGui 里 AddImage() 自绘。
- 这条路线有一批已经确认过的数据，仍有参考价值（尤其 5000E520 / 5000E640 / 5000B960 的职责分工）。
- 但用户当前明确目标是“游戏文字接管”，不是 atlas 导出成品。

1.3 明确不要的东西 [已确认]
- 不再用 glyph replay / atlas 作为最终方案
- 不再用 INT3/VEH 作为最终方案
- 不再 patch maplestory.004382EA
- 不再把 callsite 当函数入口 detour 去做 E9 jmp trampolines

============================================================
2. 稳定工程基座
============================================================
2.1 当前稳定基础
- 你原始 hook.cpp 的 Present + ImGui 覆盖层是稳定的。
- 文字原始链路：DrawOutlinedText -> ImDrawList::AddText
- 默认字体初始化：AddFontFromFileTTF(... msyh.ttc ...)

2.2 当前真正需要接管的，只是文字层
- D3D9 Present hook / WndProc / ImGui 总体框架不要乱改
- 问题核心不是 D3D9，而是文字链调用点、对象生命周期、target 裁剪

============================================================
3. 关键地址总表
============================================================
3.1 文字链核心地址 [已确认]
- 50009B80  : 高层 wrapper，最终会调用 50018890 和 5000E640
- 50009C1C  : 真正的 callsite，原始指令是 call 5000E640
- 5000E640  : 整串文本绘制/排版主函数（当前主接管点）
- 5000E520  : 单字 glyph/advance 核心函数（只做理解，不再最终 hook）
- 5000B960  : glyph 提交到目标的更底层绘制函数
- 50018890  : VARIANT/alpha/辅助对象转换函数
- 004382EA  : 上层 wrapper 内 call eax（EAX=50009B80），错误 patch 点

3.2 关键对象头 / vtbl 头 [已确认]
- textCtx 头：
  - [0x00] = 0x50025B14
  - [0x04] = 0x50025B10
- target 头：
  - [0x00] = 0x50025888
  - [0x04] = 0x50025880
- 其他相关：
  - 50025A4C / 50025AA8 / 50025414 等经常出现在 textCtx/target 内部字段

3.3 历史 atlas 路线地址 [历史备份]
- 5000BBC0 : glyph miss 时生成单字 glyph
- 5000BD20 : 16bpp atlas 写入
- 5000E9D8 / 5000E9C9 : 旧时确认过能抓整串 UTF-16 技能名的点

============================================================
4. 函数伪代码 / 调用约定（当前最重要）
============================================================
4.1 sub_50009B80 [已确认]
原型（伪代码）：
int __stdcall sub_50009B80(int a1, int a2, int a3, int a4, int a5, VARIANTARG pvarSrc, VARIANTARG a7, _DWORD *a8)

关键逻辑：
- a5 为空时直接失败
- v10 = sub_50018890(&pvarSrc, 255)，并 clamp 到 <=255
- v11 = sub_50018890(&a7, 0)
- sub_5000E640(a5, a2, a3, a4, v10, v11)
- a5 就是 textCtx

意义：
- 50009B80 是高层 wrapper，不是最终 patch 点
- 004382EA 调的其实是它，不是 5000E640

4.2 sub_5000E640 [已确认]
伪代码核心：
void __thiscall sub_5000E640(int this, int a2, char *a3, int a4, _WORD *a5, int a6, _DWORD *a7)

已确认职责：
- this = textCtx
- a2 = drawState / target state
- a3 = x（局部坐标 / pen 相关）
- a4 = y
- a5 = UTF-16 字符串
- a6 = alpha / 颜色缩放相关
- a7 = 另一个与 pen/base / Variant 相关的对象

最关键一句：
- v15 = *(_DWORD *)(this + 56)   // 即 textCtx + 0x38
- sub_5000B960(v8, a4, v15, v23, 0, 0, ...)

关键结论：
- 决定最终绘制目标 / 裁剪目标的，是 textCtx + 0x38
- 不是单纯 x/y，也不是外部长期保存的 drawState 变量

4.3 sub_5000E520 [已确认]
可理解为：
int __thiscall GetGlyph(FontCache* this, wchar_t ch, Rect* outRectOrNull)

已确认：
- 返回 EAX = advance
- 非空 out buffer 时可写 glyph rect
- 最终方案不要再 hook 它；它只是理解链路的底层依据

4.4 真正 callsite：50009C1C [已确认]
反汇编：
50009C1C  E8 1F4A0000  call canvas_m.5000E640

这才是当前真正应该 patch 的点。

============================================================
5. 真正 callsite 的现场参数（非常重要）
============================================================
5.1 有效现场 1：技能文字 [已确认]
地址：50009C1C

寄存器/栈：
- ECX = 32727DEC   ; textCtx
- [ESP+00] = 3DE9E29C ; drawState
- [ESP+04] = 0x32     ; x
- [ESP+08] = 0x60     ; y
- [ESP+0C] = L"魔法抗性"
- [ESP+10] = 0xFF     ; alpha
- [ESP+14] = 0x00     ; a6

5.2 有效现场 2：另一条文本 [已确认]
地址：50009C1C

寄存器/栈：
- ECX = 316D1D74
- [ESP+00] = 1E3E1F24 ; drawState
- [ESP+04] = 0x06     ; x
- [ESP+08] = 0x07     ; y
- [ESP+0C] = L"你的旅游怎么样"
- [ESP+10] = 0xFF
- [ESP+14] = 0x00

关键结论：
- 5000E640 是正常 thiscall 风格：ECX=textCtx，剩余参数走栈
- 之前把 EDX 当作正式调用约定的一部分，是误判

============================================================
6. 正确 / 错误 patch 方式
============================================================
6.1 错误 patch 点 [已证伪]
- 004382EA 不是最终 callsite，只是 wrapper
- 任何基于 004382EA + [ebp+..] 的取参方式都作废

6.2 错误 detour 方式 [已证伪]
- 不能用 SmartSafeHook 那种“函数入口 E9 detour + trampoline”去处理 50009C1C 这种 callsite
- 不能把 call 指令改成 jmp 指令

6.3 正确的 callsite patch 方式 [已确认]
- 原始：50009C1C = call 5000E640
- 正确：把它改成 call stub
- 进入 stub 时，栈必须保持原始形状：
  - [esp+00] = 50009C21 （原始返回地址）
  - [esp+04] = drawState
  - [esp+08] = x
  - [esp+0C] = y
  - [esp+10] = text
  - [esp+14] = alpha
  - [esp+18] = a6
  - ecx      = textCtx

6.4 stub 的正确写法 [已确认]
- stub 里只允许裸采样（mov 到全局 POD）
- 然后直接 jmp 5000E640

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VividVVO) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
