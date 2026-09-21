# 密码键盘主题创建器 & 通用刷写器 实现计划（中文版）

> **面向智能体执行者：** 所需子技能：建议使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 来逐任务实现本计划。步骤采用复选框（`- [ ]`）语法进行跟踪。

**目标：** 为所有 `.passthm` 压缩包实现通用解析（修复如 MinePass 中缺失数字的问题），并在 AirCard 中创建一个内置的「密码键盘主题创建器」，支持海报切片与逐键自定义图标。

**架构：** Python 后端（`aircard_backend.py`）使用通用的「数字 / 子文本」提取矩阵来解析主题压缩包，确保所有 iOS 18/17/16 语言环境都能找到匹配的缓存文件。在 `AirCardApp.swift` 中，原生 macOS SwiftUI 主题创建器提供可交互的 3×4 网格切片与单独按键图标指定，并支持直接刷写与 `.passthm` 导出。

**技术栈：** Python 3、Swift 5.9、SwiftUI、AppKit / CoreGraphics、ZIP 打包、macOS Sequoia / Darwin。

---

### 任务 1：在 `aircard_backend.py` 中修复通用密码刷写器

**文件：**
- 修改：`aircard_backend.py`
- 测试：`tests/test_backend_passthm.py`

- [ ] **步骤 1：编写失败的单元测试**

创建 `tests/test_backend_passthm.py`，验证 `extract_passthm_items` 能为数字 0–9 从 `MinePass_Nightly.passthm` 与 `тцк.passthm` 同时生成全部所需的 `en-` 与 `other-` 文件。

```python
import sys
from pathlib import Path

# 将项目根目录加入 sys.path
sys.path.insert(0, str(Path(__file__).resolve().parent.parent))

from aircard_backend import parse_passthm_archive, KEYPAD_SUBTEXTS

def test_minepass_nightly_extraction():
    minepass_path = "/Users/mak5er/Downloads/MinePass_Nightly.passthm"
    items = parse_passthm_archive(minepass_path, "TelephonyUI-10")
    
    # 验证数字 0 到 9 均存在
    digits_found = set()
    leaves = [item[1] for item in items]
    
    for d in range(10):
        digit_str = str(d)
        has_en = any(f"en-{digit_str}-" in l for l in leaves)
        has_other = any(f"other-{digit_str}-" in l for l in leaves)
        assert has_en, f"Missing en- variant for digit {digit_str}"
        assert has_other, f"Missing other- variant for digit {digit_str}"
        digits_found.add(digit_str)
        
    assert len(digits_found) == 10
    print("✓ MinePass_Nightly parsed all 10 digits successfully")

if __name__ == "__main__":
    test_minepass_nightly_extraction()
```

- [ ] **步骤 2：运行测试以确认其失败**

运行：
```bash
python3 tests/test_backend_passthm.py
```
预期：失败（`ImportError` 或 `AssertionError`）。

- [ ] **步骤 3：在 `aircard_backend.py` 中实现通用主题提取**

定义 `KEYPAD_SUBTEXTS` 与 `parse_passthm_archive(passthm_path, telephony_ver)`：
- 使用正则提取数字：`r'(?:^[a-zA-Z]+-)?([0-9*#])(?:-([^-\n]+))?'`
- 若缺失子文本则映射到标准子文本。
- 生成完整矩阵：
  - `en-{digit}-{subtext}--white.png`
  - `en-{digit}---white.png`
  - `other-{digit}-{subtext}--white.png`
  - `other-{digit}---white.png`
- 在 `cmd_flash_passthm` 与 `cmd_inspect_passthm` 中使用 `parse_passthm_archive`。

- [ ] **步骤 4：运行测试以确认其通过**

运行：
```bash
python3 tests/test_backend_passthm.py
```
预期：通过（`✓ MinePass_Nightly parsed all 10 digits successfully`）。

- [ ] **步骤 5：提交**

```bash
git add aircard_backend.py tests/test_backend_passthm.py
git commit -m "fix(backend): universal passcode theme parsing for all locales and archives"
```

---

### 任务 2：在 Swift 中实现键盘切片与主题导出引擎

**文件：**
- 修改：`AirCardApp.swift`

- [ ] **步骤 1：在 `AirCardApp.swift` 中实现 `KeypadSlicer` 与 `PasscodeThemeExporter`**

添加工具类 / 结构体：
- `KeypadSlicer.slicePoster(image: NSImage, zoom: Double, offset: CGPoint) -> [String: NSImage]`：
  - 以 300×300 像素渲染 10 个圆形裁切（数字 "0"…"9"），圆外为透明 alpha。
  - 匹配 3×4 iOS 拨号盘间距比例。
- `PasscodeThemeExporter.exportTheme(keys: [String: NSImage], targetURL: URL) throws`：
  - 生成 `TelephonyUI-10/` 文件夹，包含完整的 `en-` 与 `other-` 文件名矩阵。
  - 添加 `TelephonyUI-10/_big` 标记。
  - 压缩为 `.passthm` zip 文件。
- `PasscodeThemeExporter.stageTemporaryTheme(keys: [String: NSImage]) -> URL?`：
  - 保存临时 `.passthm` 包以便直接刷写。

- [ ] **步骤 2：验证切片与导出可编译**

运行：
```bash
swiftc -parse AirCardApp.swift
```
预期：通过（无语法或类型错误）。

- [ ] **步骤 3：提交**

```bash
git add AirCardApp.swift
git commit -m "feat(keypad): add KeypadSlicer and PasscodeThemeExporter engine"
```

---

### 任务 3：在 `AirCardApp.swift` 中构建密码主题创建器界面

**文件：**
- 修改：`AirCardApp.swift`

- [ ] **步骤 1：向 `AppViewModel` 添加状态变量与模型**

- 添加 `passcodeTabMode: PasscodeTabMode = .applyTheme`
- 添加 `creatorSubMode: CreatorSubMode = .posterSlice`
- 添加 `creatorPosterImage: NSImage?`
- 添加 `creatorPosterZoom: Double = 1.0`
- 添加 `creatorPosterOffset: CGPoint = .zero`
- 添加 `creatorCustomKeys: [String: NSImage] = [:]`
- 添加方法：
  - `sliceCurrentPoster()`
  - `setIndividualKeyImage(digit: String, image: NSImage)`
  - `clearCreator()`
  - `flashCreatedTheme()`
  - `exportCreatedTheme(to: URL)`

- [ ] **步骤 2：实现主题创建器视图组件**

在 `AirCardApp.swift` 中：
- 顶部分段：`Picker("", selection: $vm.passcodeTabMode) { ... }`，含 `[应用 .passthm]` 与 `[主题创建器]`。
- 在 `themeCreatorView` 中：
  - 子模式选择器：`[海报切片] | [单独按键]`。
  - 在 `海报切片` 中：
    - 图片拖放区 /「选择海报图片…」按钮。
    - 缩放滑块（`0.5x` 至 `3.0x`）、「重置位置」按钮。
    - 可交互 3×4 键盘预览，显示切片裁切效果。
  - 在 `单独按键` 中：
    - 可交互 3×4 键盘网格。每个圆形按钮都是独立的拖放目标，且可点击为对应数字选择图片。
  - 底部操作栏：
    - 「清除全部」按钮（重置创建器）。
    - 「导出 .passthm…」按钮（弹出保存面板）。
    - 「刷写到 iPhone」按钮（醒目，开始刷写）。

- [ ] **步骤 3：编译并验证界面结构**

运行：
```bash
./build.sh
```
预期：成功编译为 `build/AirCard.app` 与 `build/AirCard.dmg`。

- [ ] **步骤 4：提交**

```bash
git add AirCardApp.swift
git commit -m "feat(ui): implement interactive Passcode Theme Creator in Chinese"
```

---

### 任务 4：端到端测试与验证

**文件：**
- 使用真实设备与文件测试：
  - `/Users/mak5er/Downloads/MinePass_Nightly.passthm`
  - `/Users/mak5er/Downloads/AyuGram Desktop/тцк.passthm`
  - 由 AirCard 创建器生成的自定义主题

- [ ] **步骤 1：通过后端测试刷写 `MinePass_Nightly.passthm`**

运行：
```bash
python3 aircard_backend.py flash-passthm 00008120-001A1D0A1EE9A01E "/Users/mak5er/Downloads/MinePass_Nightly.passthm" TelephonyUI-10
```
预期：全部 10 个数字刷写成功（不缺失 2–9 键）。

- [ ] **步骤 2：部署更新后的 App 到 `/Applications/AirCard.app`**

运行：
```bash
rm -rf /Applications/AirCard.app && cp -R build/AirCard.app /Applications/AirCard.app && xattr -cr /Applications/AirCard.app
```
预期：App 流畅运行，两个标签页均加载，可在「应用 .passthm」与「主题创建器」间切换。

- [ ] **步骤 3：最终提交与清理**

```bash
git add .
git commit -m "chore: finalize Passcode Theme Creator and verified build v1.2"
```
