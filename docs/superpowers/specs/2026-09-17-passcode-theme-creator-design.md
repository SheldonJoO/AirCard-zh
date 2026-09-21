# AirCard 密码键盘主题创建器 & 通用刷写器 设计文档（中文版）

## 1. 概述
本设计文档详细说明了以下部分的架构、数据模型、UI 组件与实现逻辑：
1. **通用密码刷写器（`aircard_backend.py`）**：修复与旧版 / 多语言主题（如带 `ru-` 前缀且子文本配置各异的 `MinePass_Nightly.passthm`）的兼容性问题，确保无论设备语言或源压缩包结构如何，都能 100% 可靠刷写。
2. **内置主题创建器（`AirCardApp.swift`）**：位于「密码键盘」标签页内的可视化编辑器，提供两种创建模式：
   - **海报切片（拼图）**：将一张壁纸 / 图片按真实的 iOS 锁屏 3×4 密码键盘几何结构切片。
   - **单独按键（逐键）**：用拖放或文件选择器为每位数字（0–9）单独自定义。
3. **操作**：通过 `airlift` 即时刷写到已连接的 iOS 设备，或导出为标准 `.passthm` zip 压缩包。
4. **本地化说明**：本中文版已将 macOS App 的界面文案与标签翻译为简体中文（代码中的英文原文作为本地化键保留）。

---

## 2. 通用刷写器修复（`aircard_backend.py`）

### 2.1 刷写失败的根因
- 类似 `MinePass_Nightly.passthm` 的压缩包中包含名为 `ru-2-A B C--white.png` 的文件，而非 `en-2-A B C--white.png`。
- 数字键 0 和 1 没有子文本字母（`ru-0---white.png`、`ru-1---white.png`），因此通用替换仅对 0 和 1 有效。
- 数字 2–9 失败，因为 iOS 会根据系统语言查找 `en-{digit}-{letters}--white.png` 或 `other-{digit}-{letters}--white.png`。

### 2.2 标准子文本表
```python
KEYPAD_SUBTEXTS = {
    "0": "+",
    "1": "",
    "2": "A B C",
    "3": "D E F",
    "4": "G H I",
    "5": "J K L",
    "6": "M N O",
    "7": "P Q R S",
    "8": "T U V",
    "9": "W X Y Z",
}
```

### 2.3 提取与文件生成矩阵
对于压缩包中每个目标数字键 `D`（0–9）且子文本字母为 `LETTERS` 的有效按钮图片：
1. 保留压缩包中的原始文件名。
2. 带字母的标准英文：`en-{D}-{LETTERS}--white.png`（当 `LETTERS` 非空时）。
3. 不带字母的标准英文：`en-{D}---white.png`。
4. 带字母的其他语言：`other-{D}-{LETTERS}--white.png`（当 `LETTERS` 非空时）。
5. 不带字母的其他语言：`other-{D}---white.png`。
6. 标准键盘子文本兜底：若压缩包无字母或字母非标准，同时生成 `en-{D}-{KEYPAD_SUBTEXTS[D]}--white.png` 与 `other-{D}-{KEYPAD_SUBTEXTS[D]}--white.png`。

目标目录：`/var/mobile/Library/Caches/{telephony_ver}`（例如 iOS 18+ 为 `TelephonyUI-10`，iOS 15–17 为 `TelephonyUI-9`）。

---

## 3. 密码主题创建器架构

### 3.1 数据模型
在 `AirCardApp.swift` 中：
```swift
enum PasscodeTabMode: String, CaseIterable, Identifiable {
    case applyTheme = "Apply .passthm"
    case themeCreator = "Theme Creator"
    var id: String { rawValue }
}

enum CreatorSubMode: String, CaseIterable, Identifiable {
    case posterSlice = "Poster Slice"
    case individualKeys = "Individual Keys"
    var id: String { rawValue }
}

struct KeypadButtonGeometry {
    let digit: String
    let letters: String
    let row: Int
    let col: Int
}
```

### 3.2 键盘布局常量
- 网格：3 列，4 行。
- 标准按键排布：
  - 第 0 行：`1`（第 0 列）、`2`（第 1 列）、`3`（第 2 列）
  - 第 1 行：`4`（第 0 列）、`5`（第 1 列）、`6`（第 2 列）
  - 第 2 行：`7`（第 0 列）、`8`（第 1 列）、`9`（第 2 列）
  - 第 3 行：`0`（第 1 列）
- 宽高比与间距与真实 iOS 锁屏拨号盘一致：
  - 按键直径：75 pt
  - 水平间距：24 pt
  - 垂直间距：18 pt
  - 网格总宽：(3 × 75) + (2 × 24) = 273 pt
  - 网格总高：(4 × 75) + (3 × 18) = 354 pt

### 3.3 切片引擎（海报切片模式）
- 用户提供一张图片（`NSImage`）。
- 切片计算：
  - 将图片缩放以适配或填满键盘包围盒。
  - 为 10 个按钮分别计算归一化中心 `(cx, cy)` 与半径 `r`。
  - 以高分辨率渲染圆形遮罩（针对 `@3x` 超视网膜屏幕为 300×300 像素）。
  - 为数字 0 到 9 生成 10 个独立的圆形 `NSImage`。
- 用户控制：
  - 拖放图片目标。
  - 缩放滑块（0.5x 至 2.5x）与偏移 X/Y 调整，或拖动重新构图。
  - 实时可交互预览，在图片上显示圆形裁切效果。

### 3.4 单独按键模式
- 3×4 网格表示。
- 每个按钮拥有独立的图片拖放目标 / 点击浏览按钮。
- 用户可为单个数字设置自定义图片，也可清除任意数字。
- 缺失的数字回退为透明或标准数字字形。

### 3.5 直接刷写与导出操作
1. **刷写到 iPhone**：
   - 将当前 10 张图片编译为临时 PNG 文件，存放于内存中或临时 `.passthm` 目录。
   - 触发 `aircard_backend.py` 中的 `cmd_flash_passthm`。
   - 利用设备连接检测，逐步更新进度条。
2. **导出 .passthm**：
   - 弹出保存面板（标题「保存密码主题」）。
   - 创建标准 zip 压缩包，包含：
     - `TelephonyUI-10/` 及所有映射的 `en-` 与 `other-` 文件。
     - `_big` 或 `_small` 标记文件。
   - 以 `.passthm` 扩展名保存。

---

## 4. UI 设计与布局（中文）

### 4.1 密码标签页头部
- 顶部分段选择器：`[应用 .passthm] | [主题创建器]`。

### 4.2 主题创建器界面
- 顶部控制栏：
  - 子模式选择器：`[海报切片] | [单独按键]`。
  - 操作：`重置 / 清除全部`、`导出 .passthm…`、`刷写到 iPhone`。
- 内容区域：
  - **海报切片**模式：
    - 左 / 上：图片导入拖放区及控件（`选择图片…`、`缩放`、`适配 / 填满`）。
    - 右 / 中：可交互的 iOS 锁屏键盘预览，通过 10 个圆形裁切无缝展示构图后的海报。
  - **单独按键**模式：
    - 完整 3×4 网格，圆形按钮。点击任意圆点打开图片选择器；将图片拖到任意圆点即立即应用到该键。
- 底部状态栏：
  - 与现有进度条及活动日志集成，实时显示刷写状态。

---

## 5. 测试与验证计划
1. **主题兼容性测试**：
   - 对 `MinePass_Nightly.passthm` 刷写 → 验证数字 0–9 全部生成 `en-` 与 `other-` 变体并成功刷写到 `TelephonyUI-10`。
   - 对 `тцк.passthm` 刷写 → 验证其依旧 100% 可用。
2. **海报切片测试**：
   - 加载任意 16:9 与 19.5:9 壁纸。
   - 验证 10 张裁切 PNG 均以 300×300、圆形、遮罩外透明的方式生成。
3. **单独按键测试**：
   - 为数字 1、2、0 指定不同图片。
   - 刷写到设备并导出为 `.passthm`。
   - 检查导出 zip 的结构，确保符合 iOS 缓存标准。
