# AirCard 🎴 中文版

> **iOS 18+ 的 Apple 钱包卡片换肤工具 & 锁屏密码键盘主题工具（无需越狱）**  
> 已通过 iOS 27 正式版测试。
> 由 `airlift` AirTraffic 同步漏洞驱动。

> [!NOTE]
> 本仓库为 [Mak5er/AirCard](https://github.com/mak5er/AirCard) 的**简体中文翻译版**，界面与文档均为中文。代码逻辑与原项目保持一致，仅新增了中文本地化层，并已将默认显示语言设为简体中文。原项目作者与许可不变。

---

## 功能特性
- 🎨 **自定义卡片皮肤：** 为 Apple Pay 与钱包卡片指定自定义图案、纹理或银行 Logo。
- 🔢 **锁屏密码主题（.passthm）：** 直接将热门 `.passthm` 主题的自定义拨号键图案应用到 iOS 18+ 锁屏。
- 🧩 **密码主题创建器：** 既可用一张壁纸切片生成（无缝海报切片），也可逐键单独制作。
- 🔍 **可交互构图：** 在拨号键内直接平移、缩放图案，并实时预览 iPhone 效果。
- ✏️ **编辑现有 .passthm 主题：** 直接在创建器中打开任意 Cowabunga 或 Nugget 主题包，调整按键图案、重新定位图片，再导出或刷写。
- ⚡ **单卡 & 批量自定义：** 可为每张卡设置不同图案，也可一键将同一设计应用到所有卡片。
- 📱 **零门槛卡片识别：** 在 iPhone 钱包 App 中轻点任意卡片，即可实时识别其哈希。
- 🚀 **100% 独立运行（通用版）：** 原生支持 **Apple 芯片** 与 **Intel (x86)** Mac，所有设备通信工具与图像引擎均已内置。
- 📦 **零前置依赖：** macOS 用户无需安装 Homebrew、Python 包或配置终端。

---

## 安装

### macOS（通用 DMG）
1. 从 [Releases](https://github.com/mak5er/AirCard/releases) 下载 **`AirCard.dmg`**。
2. 打开 `AirCard.dmg`，将 **`AirCard.app`** 拖入 **应用程序** 文件夹。
3. 完全兼容 **Apple 芯片** 与 **Intel (x86)** Mac。

> [!NOTE]
> **macOS 首次启动（Gatekeeper）：**
> 若首次启动时 macOS 弹出「无法验证开发者」提示：
> - **方法一（图形界面）：** 在「应用程序」中右键（或 Control 单击）`AirCard.app` ➔ 点击 **打开** ➔ 再次点击 **打开**。
> - **方法二（终端）：**
>   ```sh
>   sudo xattr -cr /Applications/AirCard.app
>   ```

---

## 如何自定义 Apple 钱包卡片
1. 用 USB 线将 iPhone 连接 Mac，并确保 iPhone 已解锁且已「信任此电脑」。
2. 在 AirCard 中停留在 **钱包卡片** 标签页，点击 **扫描卡片**。
3. 在 iPhone 上：
   - **双击侧边（电源）按钮** 打开 Apple Pay。
   - 通过 **Face ID** 验证。
   - **轻点你的卡片**（或再轻点一次）即可触发实时识别！
4. 点击任意卡片模型，或将图片直接拖放到卡片上。
5. 点击 **刷写皮肤**。
6. 在 iPhone 上从多任务界面强制关闭 **钱包** App（或重启）即可看到新的自定义卡片样式！

---

## 如何应用锁屏密码主题（.passthm）
1. 在 AirCard 顶部切换到 **密码键盘主题** 标签页。
2. 将任意 `.passthm` 文件拖入 App（或点击 **选择 .passthm 文件**）。
3. AirCard 会解析主题，并在数字拨号盘（0–9、*、#）上显示可交互预览。
4. 点击 **应用密码主题**。
5. 重启 iPhone 以重新加载锁屏缓存，即可看到自定义密码按键！

> [!TIP]
> **通用语言与加粗文字支持：**  
> AirCard 会自动为所有系统语言（英语、乌克兰语、俄语、西班牙语、德语、法语等）扩展并刷写自定义拨号键素材，同时生成标准与 **加粗文字** 缓存位图（`--white` 与 `--white-bold`），无论你的 iOS 语言或辅助显示设置如何，主题都能生效！

---

## 从源码构建

```sh
git clone https://github.com/mak5er/AirCard.git
cd AirCard
chmod +x build.sh
./build.sh
```
此命令会构建通用二进制（`arm64` + `x86_64`），将依赖打包进 `build/AirCard.app`，并输出 `build/AirCard.dmg`。

> 本中文版的本地化资源位于 `Localization/zh-Hans.lproj/Localizable.strings`，由 `build.sh` 自动打包进 `.app` 的 `Resources/zh-Hans.lproj/` 目录，并将 `Info.plist` 的默认显示语言设为 `zh-Hans`。

---

## 贡献者
- **[@mak5er](https://github.com/mak5er)**（开发者）— [GitHub](https://github.com/mak5er) · [Twitter / X](https://x.com/mak5er)
- **[@Lumid-Off](https://github.com/Lumid-Off)**（贡献者与开发者）— [GitHub](https://github.com/Lumid-Off) · [Twitter / X](https://x.com/LumidOff)
- **[AirLift](https://github.com/0xjohnnydev/airlift)** by **[0xjohnny (@0xjohnnydev)](https://github.com/0xjohnnydev)**：`AirliftFFI` 所基于的原始 AirTraffic/ATAirlock 沙盒逃逸概念验证。

## 致谢
- 核心漏洞基于 `airlift`（AirTraffic 同步逃逸）。

---

## 支持原作者

本中文版是纯粹的本地化分支，**不做任何捐赠收款**。所有功劳都属于上游作者。

如果你觉得 AirCard 有用，请到**原项目**去支持他们的后续开发：

- 上游仓库：[github.com/mak5er/AirCard](https://github.com/mak5er/AirCard)
- 开发者：[@mak5er](https://github.com/mak5er) · [@Lumid-Off](https://github.com/Lumid-Off)

> 本仓库已移除原 README 中的全部捐赠渠道（PayPal / TON / USDT）。需要支持作者请走上游，不要向本分支的任何地址付款。

---

## 许可
本项目沿用原项目许可，详见仓库中的 `LICENSE` 文件。
