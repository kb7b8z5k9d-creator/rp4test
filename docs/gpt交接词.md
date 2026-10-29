# RP4 Pro 专用 DroidDeck 整合优化项目——DeepSeek 工程交接书

## 一、你的身份与任务

你是一名负责 Android、Linux、Vulkan、Mesa、Wine/Proton、FEX 及跨平台编译的工程开发代理。

我希望你借助 AI 编程能力，将五个现有开源项目中适用于 Retroid Pocket 4 Pro 的有效技术成果进行选择性整合，开发一套兼容性、稳定性和性能相对最优的 **RP4 Pro 专用 DroidDeck**。

**注意：你没有任何此前的对话背景。以下内容是完整交接资料，请先阅读，再独立访问 GitHub 核对。**

这不是让你把五个仓库机械合并。

我需要的是：

1. 选择可靠、已经通过实机验证的项目作为基础。
2. 移植其他项目中能够改善 RP4 Pro 的补丁。
3. 保留可回退的稳定基线。
4. 编译生成自己的 Android APK。
5. 在 USB 调试模式下，通过 ADB 对真实 RP4 Pro 进行测试。
6. 根据实际日志修复问题。
7. 最终得到一套可独立编译、测试、维护和升级的定制版本。

项目追求的是实际游戏运行效果，不是简单追求最新版本号或代码量。

---

## 二、目标硬件：Retroid Pocket 4 Pro

这是本工程唯一需要优先支持和优化的设备。

| 参数 | 具体配置 |
|---|---|
| 设备 | Retroid Pocket 4 Pro，简称 RP4 Pro |
| SoC | MediaTek Dimensity 1100 |
| GPU | ARM Mali-G77 MC9 |
| CPU | 4× Cortex-A78 + 4× Cortex-A55 |
| RAM | 8GB |
| 系统 | Android 13 |
| Linux 内核 | 4.19.191 |
| 原厂 Vulkan | Vulkan 1.1.177 |
| 原厂 ARM GPU 驱动 | r32p1，曾报告 `v1.r32p1-01eac0` |
| 内存接口 | 旧内核 ION 机制，缺少 `/dev/dma_heap` |
| 显示 | 约 1334×750，60Hz |
| 调试连接 | USB ADB |
| 是否 Root | 默认无 Root，不允许假设具备 Root |

这些参数来自已有项目测试记录，但你仍需在实机上用 ADB 重新核验。

**重点：不要把 RP4 Pro 当成普通的 Mali-G77 设备。**

即使 GPU 型号相同，不同 Android GPU 驱动版本、Linux 内核、内存管理机制也会导致完全不同的兼容性。

目前最大的硬件限制可能并非 GPU 算力，而是：

- 原厂 Vulkan 仅 1.1。
- ION/KBASE 内存共享兼容性。
- Gamescope、Android 合成器及 Linux 客户端之间的图像共享。
- FEX 的 x86 到 ARM64 转译开销。
- Vulkan 驱动特性与 DXVK/VKD3D-Proton 的需求不匹配。
- 8GB 内存下 Android 低内存回收机制的影响。

---

## 三、五个开源项目的完整介绍

以下版本信息核对于 **2026 年 10 月 29 日**。

这些是交接基线，不代表以后不会更新。

开始工作时，必须重新检查 GitHub，并记录每个仓库实际使用的 Commit SHA。

### 项目一：DroidDeck 官方项目

仓库：

https://github.com/Droid-Deck/DroidDeck

官方 Release：

https://github.com/Droid-Deck/DroidDeck/releases

官方 main：

https://github.com/Droid-Deck/DroidDeck/tree/main

**定位：整个运行环境的上游基础项目。**

DroidDeck 是让 Android 设备本地运行 Steam 客户端和 PC 游戏的开源项目。

它不是 Steam 串流，也不是传统的 Android 原生游戏模拟器。

它使用 Android 上的 Linux 运行环境，配合 Steam、ARM64 Proton、FEX、Wine、DXVK、VKD3D-Proton、Gamescope 等组件，在 ARM 设备上运行 PC 游戏。

其主要组成部分包括：

- Android 应用及生命周期管理。
- Linux 用户空间和运行时。
- Steam 客户端启动、登录及管理。
- ARM64 Proton / Wine。
- FEX x86 转译。
- Vulkan / OpenGL 图形运行环境。
- Gamescope 和图像合成器。
- 控制器与 Steam Input。
- 游戏库、兼容层及组件管理。
- 性能 HUD、帧生成等附加功能。

已知正式版为 `0.6.0`，主分支仍快速更新。

**优势：**

- 体系完整。
- 社区规模最大。
- 通用功能与框架维护能力强。
- Proton、FEX、系统组件等上游改进较多。
- 适合作为长期技术来源。

**缺点：**

- 正式版本主要面向支持的 Adreno 设备。
- RP4 Pro 的 Mali-G77 尚未获得完整、正式的兼容支持。
- 不应假设官方最新版可以直接在 RP4 Pro 上运行 Steam。

**对本工程的作用：**

重点挑选真正有价值的通用修复，而不是无条件把最新 main 合并到 RP4 Pro 版本。

特别关注：

- FEX / Proton 运行稳定性。
- CPU 转译和系统调用开销。
- Gamescope。
- 内存与资源回收。
- 控制器和音频。
- 游戏启动、退出、休眠恢复。
- 日志与崩溃诊断。

普通 UI 更新暂不作为重点。

### 项目二：ASEbrahim MaliBridge

仓库：

https://github.com/ASEbrahim/MaliBridge

Release：

https://github.com/ASEbrahim/MaliBridge/releases

RP4 Pro 兼容性 Issue：

https://github.com/ASEbrahim/MaliBridge/issues/10

**定位：通过 Android 原厂 Mali Vulkan 驱动运行 DroidDeck 的主要技术路线。**

它不是一个完全独立于 DroidDeck 的模拟器。

ASEbrahim 在 DroidDeck 基础上加入 MaliBridge，通过 Vulkan 命令转发及兼容层，利用 Android 系统现有的 Mali GPU 驱动。

与直接使用 Linux Mesa PanVK 不同，它主要依靠手机原厂 ARM Vulkan 驱动。

截至交接时，最新官方版本是：

`v0.5.4.2`

此版本已经针对旧 Mali GPU、Steam 启动黑屏、SD 卡游戏库及旧 Vulkan 驱动等问题进行修复。

**RP4 Pro 的已知实测：**

在 `v0.5.4.1` 中，已有 RP4 Pro 用户报告：

- Steam Big Picture 可以启动。
- 《Fields of Mistria》可以运行。
- 《Megabonk》可以运行。
- Steam 菜单可能较慢。
- 仍有画面闪烁和伪影。

这证明 MaliBridge 不是纯理论路线，而是已经在目标设备上获得实际成果。

**优势：**

- 已有 RP4 Pro Steam 成功记录。
- 对原厂 Vulkan 1.1 GPU 做了兼容处理。
- 具有持续发布和维护机制。
- 可以借用原厂 GPU 驱动已经存在的功能。
- 目前比 PanVK Linux 路线更接近 RP4 Pro 实用化。

**限制：**

- 原厂驱动本身的错误需要逐项处理。
- Vulkan 1.1 的旧驱动无法天然满足所有现代 DXVK/VKD3D 要求。
- 向 Gamescope 呈现 Vulkan 1.2，不等于游戏本身也获得完整 Vulkan 1.2/1.3。
- 某些新功能仍可能需要 Vulkan 特性模拟、驱动错误规避或着色器变换。

**本项目作用：**

它是优先使用的 GPU 兼容层。

应优先将其官方新版本中经过验证的有效改进整合进最终版本。

### 项目三：yujikaido 的 DroidDeck RP4 Pro 专项版

仓库：

https://github.com/yujikaido/droiddeck-rp4pro

完整应用源码：

https://github.com/yujikaido/droiddeck-rp4pro/tree/source

Release：

https://github.com/yujikaido/droiddeck-rp4pro/releases

调试记录：

https://github.com/yujikaido/droiddeck-rp4pro/blob/main/docs/PROGRESS-LOG.md

MaliBridge 上游 PR：

https://github.com/ASEbrahim/MaliBridge/pull/35

**定位：专门针对 RP4 Pro 进行深入修复的第三方定制分支。**

这是本工程目前最重要的基线候选。

该开发者使用自己的 RP4 Pro，通过 ADB、Vulkan 日志、驱动分析、着色器分析等方法修复了多个具体问题。

其开发过程有 Claude Code 参与，设备测试、方向选择由开发者负责。

截至交接时，最新版本为：

`fix43`

注意：fix14 到 fix43 不是公开连续发布了 29 个版本。已公开的主要 Release 包括 fix13、fix14、fix42、fix43。

**版本演进：**

- fix13：解决部分 G77 特定问题，支持 Steam 与 PC 游戏运行。
- fix14：增加 Vulkan 1.1 下 LSFG 帧生成相关修复。
- fix42：基于 MaliBridge v0.5.4.1 源码完整编译，增加 Retro 模拟器前端。
- fix43：加入指向其自身 GitHub Release 的应用更新通道。

与早期替换官方 APK 内部动态库不同，fix42 之后已经具备完整应用源码。

**重要的技术修复：**

第一，Mali-G77 r32p1 拒绝某些带有 `VK_IMAGE_CREATE_MUTABLE_FORMAT_BIT` 的图像格式查询。

结果是 Gamescope 找不到可使用的 DRM modifier，导致 Steam 启动失败。

修复方式是对特定失败查询进行兼容性重试。

第二，AHardwareBuffer 图像的内存要求与布局查询存在原厂驱动兼容问题。

项目通过相应的图像代理和兼容处理，避免触发原厂驱动错误。

第三，《伊苏：起源》中存在顶点计算产生 NaN 的情况。

原厂 Mali 驱动可能把这些无效顶点绘制成覆盖整个房间的白色图像。

项目增加 `nan_position` 修复，对异常顶点位置进行处理。

第四，Android SD 卡使用 `noexec` 文件系统挂载。

部分 Wine ARM64EC 进程尝试执行 SD 卡上的组件时发生失败。

项目修改相关 preload 逻辑，以兼容这类场景。

第五，帧生成在切换窗口大小、重新启动游戏时，可能提前释放 GPU 仍然使用的资源。

项目修复了相关同步和资源生命周期问题。

第六，增加 Vulkan 1.1 下 LSFG 的特定扩展路径。

**在 RP4 Pro 上已公布的 Steam PC 游戏实测：**

| 游戏 | 已公开测试表现 |
|---|---|
| 《伊苏：起源》Ys Origin | 普通房间约 60–69 FPS，特殊图形场景约 50–60 FPS |
| 《伊苏 I&II 编年史》 | 经过 FEX 配置调整后约 60 FPS |
| 《忍者神龟：施莱德的复仇》 | 约 60 FPS |

这些是开发者的实机报告，不等于所有用户或所有版本都能完全复现。

**优势：**

- 直接针对目标设备。
- 已经跑通 Steam PC 游戏。
- 对 G77 的图形缺陷有具体分析。
- 有完整调试记录。
- 已经具备可编译的完整源码。
- 开发速度较快。

**缺点：**

- 主要只有一台 RP4 Pro 的测试记录。
- 较新的功能缺乏大规模回归测试。
- 由个人维护。
- 自行签名，与 ASEbrahim 官方 APK 不可直接互相覆盖。
- 与上游分支长期分化可能增加维护负担。

**上游合并情况：**

MaliBridge PR #35 尚未合并，但 ASEbrahim 已表示正在自行吸收相关 G77 修复。

提交给 DroidDeck 官方的 PR #1692 和 #1693 已关闭，原因涉及该项目的 AI 共同作者署名检查。不能假设这两项补丁已进入官方 main。

**本项目作用：**

优先将其完整源码作为第一版可运行基线。

但不要无条件继承所有附加功能，也不要把个人分支已有的修复再次重复移植。

### 项目四：The412Banner 非骁龙 DroidDeck 实验路线

源码分支：

https://github.com/Droid-Deck/DroidDeck/tree/test/non-adreno-all

实验 APK：

https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases

相关 RP4 Pro 测试：

https://github.com/The412Banner/droiddeck-nonadreno-drivers/issues/7

**定位：DroidDeck 的非 Adreno GPU 多路线实验集成。**

它尝试在非骁龙设备上支持：

- MaliBridge。
- Mesa PanVK。
- Venus。
- 不同 Android 显示驱动。
- 不同 Linux Runtime 驱动。
- Gamescope 兼容处理。
- GPU 驱动自动选择。

截至交接时，最新 APK 构建为：

`948b9cbc`

内置 MaliBridge v0.5.4。

**优势：**

- 涉及不同 GPU 驱动路线。
- 包含针对非 Adreno 的驱动自动选择逻辑。
- 对 Gamescope 图像共享问题开展过大量测试。
- 具备跨设备测试数据。

**限制：**

- 不应把其他 Mali 机型的成功当成 RP4 Pro 的成功。
- 当前实验版本尚无充分证据说明可在 RP4 Pro 上稳定运行 Steam。
- 某些 PanVK 方案在 RP4 Pro 上仍存在内存共享问题。

以前的 RP4 Pro 实测中出现过：

`ION_IOC_ALLOC failed: Inappropriate ioctl for device`

`vkAllocateMemory failed`

`KBASE_IOCTL_MEM_IMPORT failed: Cannot allocate memory`

`VK_ERROR_DEVICE_LOST`

这些错误说明内存分配及缓冲区共享链路尚未打通。

**本项目作用：**

不是作为基础，而是选择性提取：

- Gamescope 修复。
- 非骁龙 GPU 检测逻辑。
- 驱动配置管理。
- 内存导入、图像呈现相关补丁。
- 其他能明确解决 RP4 Pro 问题的代码。

不要整包覆盖已经运行成功的 MaliBridge 环境。

### 项目五：faux123 PanVK

仓库：

https://github.com/faux123/panvk-faux123-driver-rp4pro

Release：

https://github.com/faux123/panvk-faux123-driver-rp4pro/releases

**定位：针对 RP4 Pro 的实验性 Mesa PanVK 驱动路线。**

它与 MaliBridge 属于不同的 GPU 解决方案。

MaliBridge 主要通过原厂 Android Vulkan 驱动工作。

PanVK 则使用 Mesa 开源 Vulkan 驱动，并借助设备现有 Mali 内核驱动接口。

截至交接时版本：

`v1.1.2`

已有 Android 驱动包：

`panvk_faux123_v1.1.2.adpkg.zip`

该版本主要修正了 Mali-G77/G57 中部分 GPU 内存分配导致 Android 回收游戏进程的问题。

**重点区别：**

该 `.adpkg.zip` 是 Android 自定义驱动包，不应直接视为可用于 DroidDeck Linux 环境的 PanVK 驱动。

DroidDeck 的 PanVK 路线通常还需要匹配的 Linux Runtime 驱动及内存共享支持。

以前的 RP4 Pro Linux 侧测试存在 ION/KBASE 问题。

**优势：**

- 可能绕过原厂旧 Vulkan 驱动的一部分限制。
- 使用较新的 Mesa 驱动实现。
- 具备研究现代 Vulkan 特性的潜力。
- 与 MaliBridge 形成不同技术路线。

**限制：**

- Android 端的 PanVK 成功不等于 Linux 端成功。
- Vulkan 1.3 的版本声明不代表所有现代游戏所需特性可用。
- 不应假设能够直接支持 DX12。
- Linux 侧内存共享与 Gamescope 兼容仍是关键障碍。

**本项目作用：**

作为第二条实验驱动路线。

不要在第一版稳定构建中强行把 PanVK 和 MaliBridge 混合。

---

## 四、最终技术目标

为避免把工程变成无限扩大范围的重构项目，优先目标依次为：

**P0：建立可独立编译、能在 RP4 Pro 上运行 Steam 的版本。**

至少保留 yujikaido 已验证的基础能力。

**P1：整合 ASEbrahim 官方新版改进。**

在不破坏既有功能的前提下升级 MaliBridge。

**P2：整合其他上游有明确收益的修复。**

包括 Gamescope、内存管理、FEX、Proton 及必要的系统组件。

**P3：提高游戏兼容性和持续运行稳定性。**

优先修复崩溃、图像伪影、卡顿及内存异常。

**P4：独立研究 PanVK 与 DX12。**

不把它作为第一阶段验收要求。

最终希望形成一个：

- 可以自己编译。
- 可以安装在 RP4 Pro 上。
- 不依赖第三方手工修改 APK。
- 有独立签名和包名。
- 保留完整源码与补丁历史。
- 可以持续同步上游。
- 能方便比较不同驱动与构建版本。
- 能通过 ADB 复现和诊断问题。

的专用 DroidDeck 版本。

---

## 五、总体工程原则

### 1. 不允许直接进行“五仓库全量合并”

五个项目并不处于同一个软件层级。

DroidDeck 是整体框架。

ASEbrahim 是完整应用分支加 MaliBridge 兼容层。

yujikaido 是设备专项分支。

The412Banner 主要是多 GPU 实验集成。

faux123 是另一条底层 Vulkan 驱动路线。

因此必须采用：

**稳定基线 + 选择性补丁移植 + 独立实验分支**

而不是将所有文件简单覆盖。

### 2. 每项改动都要有明确的理由

每一个移植补丁都要说明：

- 原始项目和 Commit。
- 解决的具体问题。
- 当前基础版本是否已经包含。
- 和现有代码是否冲突。
- 涉及哪些文件和运行组件。
- 理论上的收益。
- 如何测试。
- 如何回退。

如果功能已经存在，不要重复移植。

如果补丁不能证明对 RP4 Pro 有意义，可以暂不整合。

### 3. 不进行未经授权的额外优化

不要因为发现其他代码可以重构，就顺手优化。

不要自动清理无关文件、替换工具链或升级所有依赖。

不要擅自修改用户其他工作区。

保持本工程的改动范围最小化。

### 4. 所有实验都要可回退

使用独立 Git 分支或可追踪提交。

保持未修改的原始基线。

任何性能优化失败都必须能恢复。

不要为了新的 Vulkan 实验破坏已能运行的 MaliBridge 路线。

---

## 六、开发环境

用户的电脑主要使用：

- Windows 11。
- VS Code。
- Python。
- Git。
- Android SDK / NDK。
- JDK 17。
- Node.js。

这些工具是否全部正确安装，需要你现场检查。

项目原有构建脚本依赖：

- Docker。
- Linux 构建环境。
- Android SDK / NDK。
- Java 17。
- Rust。
- Python 3。
- Git。
- GitHub CLI。
- zstd 等工具。

可以优先考虑 Windows 11 + WSL2 + Docker 的构建方案。

但需要注意，Linux 构建脚本不能直接把 Windows 可执行文件当作 Linux SDK 工具使用。

请先检查：

- WSL2 是否存在。
- Docker 是否可用。
- Linux SDK / NDK 是否可用。
- Android SDK 和 JDK 的具体版本。
- 可用磁盘空间。
- 是否已存在相关源码。
- ADB 是否能访问 RP4 Pro。

不得未经用户同意修改全局工具链、安装需要管理员权限的软件或删除原有开发环境。

所有新增项目文件集中放在用户指定的工程工作目录中。

### 构建基础

yujikaido 的 `source` 分支包含：

`tools/build_local.sh`

可以参考该脚本配置完整构建。

源码使用 Gradle。

可以通过：

`DROIDDECK_BUILD_VARIANT=debug`

选择调试 APK 构建。

其 Gradle 配置还支持：

`DROIDDECK_APPLICATION_ID`

`DROIDDECK_APP_ID_SUFFIX`

`DROIDDECK_APP_LABEL`

等应用包名和名称参数。

**不要直接照抄参数后假设构建会成功，必须阅读当前源码确认。**

建议最终测试版采用独立应用 ID，例如：

`com.rp4pro.droiddeck.lab`

并使用明确的显示名称，例如：

`RP4Pro Lab`

要检查包名变化是否影响 Steam 启动、Android Provider、更新逻辑和其他应用组件。

尤其需要检查默认更新通道，确保自行构建的实验版本不会自动安装其他开发者签名的 APK。

---

## 七、具体实施阶段

### 阶段 A：源码调查与版本冻结

此阶段不修改用户设备。

首先分别获取五个项目的源码信息。

记录：

- GitHub 仓库地址。
- 当前 Release。
- 当前分支。
- Commit SHA。
- 子模块 Commit。
- 关键构建资产版本。
- 许可证。
- 当前已知兼容性问题。

注意：The412Banner 的测试 APK 使用滚动 Release 标签，不能只比较标签名，必须检查 APK 文件名和资产更新时间。

建立一张补丁对照表，至少包含：

- yujikaido 已修复的问题。
- ASEbrahim 最新官方版新增的修复。
- 两者重复的代码。
- DroidDeck 官方值得提取的补丁。
- The412Banner 中对 G77 有意义的补丁。
- PanVK 尚未解决的技术问题。

尤其检查 MaliBridge PR #35 是否已经合并，以及官方版是否已有 `nan_position` 或等效修复。

不要重复开发已经解决的问题。

阶段成果：形成准确、可复核的整合清单。

### 阶段 B：独立编译 yujikaido 基线

建议首先以：

`yujikaido/droiddeck-rp4pro` 的 `source` 分支

为基础。

仓库：

https://github.com/yujikaido/droiddeck-rp4pro

注意：

该仓库的 `main` 主要包含文档、测试证据和补丁说明；完整应用源码在 `source` 分支。

源码还包含 MaliBridge 子模块，必须检查其具体指向的提交，不能简单初始化后就自动使用官方最新 main。

任务：

1. 下载并固定 source 分支的确定提交。
2. 初始化必要子模块。
3. 检查构建脚本。
4. 检查签名设置。
5. 检查更新来源。
6. 构建原始基线 APK。
7. 验证 APK 的包名、版本和签名。
8. 保存构建日志、哈希及实际使用的源码版本。

**此阶段不进行任何整合优化。**

先证明当前环境可以可靠复现基础构建。

如果无法编译，需要先定位具体原因。

不要通过删除复杂组件或大规模修改源码来掩盖构建问题。

### 阶段 C：USB ADB 基线测试

当基线 APK 构建成功后，连接 RP4 Pro。

先检查：

`adb devices -l`

确认目标设备唯一，且 USB 调试已获得授权。

建议记录以下只读信息：

`adb shell getprop ro.product.model`

`adb shell getprop ro.build.version.release`

`adb shell uname -a`

`adb shell cat /proc/meminfo`

还要通过可用的 Vulkan 设备报告工具，核实实际 Vulkan 驱动信息。

**安装要求：**

- 必须先确认测试 APK 使用独立包名。
- 不得卸载用户现有的 DroidDeck、Eden 或其他游戏应用。
- 不得覆盖其他开发者签名的 APK。
- 不得清除用户应用数据。
- 不得自动修改 Android 系统安全设置。
- 不得 Root 或刷机。
- 不得修改设备分区。

如果应用必须更改系统设置才能运行，先说明具体原因、风险和恢复办法，得到用户同意后再操作。

安装测试 APK 可以使用：

`adb install -r <测试APK路径>`

但只允许安装已确认属于本工程的独立测试包，不允许覆盖用户原有应用。

Steam 登录凭据必须由用户亲自输入。

不要读取、导出或上传 Steam Cookie、令牌、密码、Steam Guard 数据。

不要自动操作 Steam 账号安全设置。

首先测试：

- APK 是否正常打开。
- 初始化是否正常。
- Steam 运行环境是否能够启动。
- 是否出现显示初始化失败。
- Steam 游戏模式是否正确显示。
- 手柄是否工作。
- 音频是否正常。
- 游戏启动、退出后能否再次启动。
- 是否能够休眠和恢复。

不要将 Steam 客户端成功启动误认为游戏兼容性测试通过。

### 阶段 D：整合最新 ASEbrahim MaliBridge

在基线通过后，再处理官方新版。

截至交接时，ASEbrahim 最新官方版本为：

`v0.5.4.2`

yujikaido fix43 的基础是：

`v0.5.4.1`

应检查两者的具体差异。

**重要：这里是选择性移植官方新版的有效改进，不是直接替换整个 MaliBridge 目录。**

必须保留 yujikaido 的：

- G77 专项修复。
- `MUTABLE_FORMAT` 查询修复。
- AHardwareBuffer 相关兼容处理。
- `nan_position`。
- 其他必要的 G77 修复。

同时检查哪些功能已被 ASEbrahim 官方实现，避免重复或冲突。

完成后重新编译和测试。

如果新版本导致游戏回归，使用 Git 定位具体改动。

优先保持已确认可用的功能。

### 阶段 E：提取 DroidDeck 官方改进

在阶段 D 稳定之后，审查官方 DroidDeck 的最新 main。

重点是：

- Proton/FEX 兼容修复。
- 游戏生命周期管理。
- Linux 系统调用性能。
- 音频和控制器。
- Gamescope。
- 崩溃诊断。
- 内存管理。
- 其他能够解决 RP4 Pro 实际问题的代码。

不要整包升级整个官方 main。

官方 main 与 MaliBridge 定制版之间可能存在应用架构及原生组件版本差异。

如果某个补丁依赖多个较新的组件，应先评估完整依赖链，再决定是否移植。

不要因为官方代码更新日期更晚，就默认其在 RP4 Pro 上更好。

### 阶段 F：提取 The412Banner 有价值的修复

研究：

`test/non-adreno-all`

以及对应测试 APK 的版本历史。

重点调查：

- Gamescope 对图像格式的兼容处理。
- DMA-BUF/DRM modifier 相关改进。
- Vulkan 设备检测。
- Mali 驱动选择。
- 图像输出和内存共享。
- 其他与 G77 相关的修复。

核对这些修改是否已经存在于 MaliBridge 或 yujikaido 分支中。

只移植独立、必要且可验证的代码。

不要同时启用多个互相竞争的图形兼容层。

如果补丁与 MaliBridge 架构冲突，先记录为实验候选，不要强制合入稳定版本。

### 阶段 G：研究 faux123 PanVK

**这一阶段与主版本隔离。**

目标是判断能否建立第二条独立可选的图形驱动路线。

重点调查此前出现的：

`ION_IOC_ALLOC failed`

`KBASE_IOCTL_MEM_IMPORT failed`

`VK_ERROR_DEVICE_LOST`

以及 GPU 内存分配和缓冲区导入问题。

需要分清：

1. Android 侧自定义 PanVK 驱动。
2. Linux Runtime 侧 PanVK。
3. Mali 内核驱动接口。
4. Android 与 Linux 进程间的内存共享。
5. Gamescope 所需的显示输出能力。

不要将 Android `.adpkg.zip` 直接当作 Linux 驱动安装。

在没有解决基础内存共享问题以前，不要求支持 DX12。

**如果这一阶段出现结构性阻碍，允许结束实验，不得牺牲稳定版。**

---

## 八、关于 Vulkan 1.3 与 DirectX 12

我特别关心 RP4 Pro 未来是否能够运行 Steam DX12 游戏。

但你必须准确区分技术层级。

Windows DX12 游戏通过 VKD3D-Proton 转换成 Vulkan。

SteamOS 在 Steam Deck 上能运行 DX12，是因为 AMD GPU 驱动提供了足够的 Vulkan 功能。

RP4 Pro 的原厂 Mali 驱动只有 Vulkan 1.1.177。

MaliBridge 可以在部分场景模拟、转换或提升 Vulkan 接口版本，但这不代表 GPU 的所有特性均已补齐。

尤其需要注意：

**Gamescope 获得 Vulkan 1.2 接口，不等于游戏进程获得 Vulkan 1.3。**

不要把软件报告的 Vulkan 版本号与真实的功能支持等同。

对 DX12 的探索顺序应当是：

1. 确认当前 DXVK/VKD3D 组件及其版本。
2. 确认 GPU 实际支持的 Vulkan 功能、扩展及限制。
3. 确认当前启动失败属于版本要求、缺失特性、内存还是驱动崩溃。
4. 判断是否可以通过 MaliBridge 的兼容处理实现。
5. 再研究 PanVK 路线是否有实质优势。

**禁止为通过启动检测而无条件伪造 Vulkan 1.3 能力。**

不能把“游戏进入标题画面”当作 DX12 兼容成功。

必须经过实际渲染和稳定运行验证。

---

## 九、实机测试与数据收集

使用 USB ADB，在 Windows 主机上收集调试信息。

可以先运行：

`adb logcat -v threadtime > rp4pro_logcat.txt`

在复现问题后停止采集。

不同设备和 Android 版本下，系统日志权限可能有限。

重点关注：

- `FATAL EXCEPTION`
- `SIGSEGV`
- `SIGABRT`
- `VK_ERROR_DEVICE_LOST`
- `vkAllocateMemory`
- `OutOfMemoryError`
- `lmkd`
- `lowmemorykiller`
- `ION_IOC_ALLOC`
- `KBASE_IOCTL_MEM_IMPORT`
- Gamescope 错误
- MaliBridge server 错误
- DXVK / VKD3D 错误
- FEX / Wine 错误

此外，DroidDeck 自身如果提供诊断日志导出，应优先结合应用日志分析，而非仅依赖 Android logcat。

**日志安全要求：**

不得直接公开上传未经检查的完整日志。

必须处理其中可能包含的 Steam 凭据、会话令牌、用户名、设备标识、私人路径等敏感信息。

### 基准测试要求

每次比较不同构建，尽量保持以下条件一致：

- 相同设备。
- 相同性能模式。
- 相同风扇策略。
- 相同游戏版本。
- 相同分辨率。
- 相同 Proton/FEX 配置。
- 相同图形设置。
- 相同存储位置。
- 相近温度及负载条件。

重点记录：

| 指标 | 目的 |
|---|---|
| Steam 启动是否成功 | 基础兼容性 |
| 游戏是否进入实际玩法 | 真实兼容性 |
| 平均 FPS | 整体性能 |
| 1% Low FPS | 卡顿情况 |
| 帧时间波动 | 流畅度 |
| RAM / GPU 内存使用 | 是否容易被 Android 杀进程 |
| CPU / GPU 负载 | 定位瓶颈 |
| 温度与降频 | 判断持续运行性能 |
| 画面错误 | 图形驱动稳定性 |
| 长时间运行是否崩溃 | 稳定性 |

如果某个指标无法可靠采集，应写明，而不是编造。

实际测试优先使用用户已经拥有并能够合法运行的游戏。

可以参考 yujikaido 的《伊苏：起源》等公开数据，但不要默认用户已经购买这些游戏。

最初应优先测试轻量游戏，确认稳定后再尝试更复杂的 3D 游戏。

建议每轮比较至少有冷启动、游戏场景、退出重启三个过程。

若条件允许，再进行重复运行和较长时间稳定性测试。

---

## 十、软件安全及签名要求

由于应用将用于登录 Steam 账号，这一部分必须认真处理。

- 只从明确的 GitHub 官方仓库获取源码和构建资产。
- 固定源码提交和依赖版本。
- 核验可用的 SHA256 或其他发布校验信息。
- 不引入未知来源的预编译动态库。
- 保留源码与最终 APK 的构建关系。
- 使用自己的签名密钥，不依赖其他开发者的个人签名。
- 禁止把私钥和密码提交到 Git。
- 不向 AI、第三方服务或日志输出提供 Steam 凭据。
- 不自动更改 Steam 登录安全机制。
- 不自动开启不安全的网络调试服务。
- 不自动发布 APK 到网络。

需要检查新包名和新签名是否影响应用自身的更新机制。

不要沿用 yujikaido 的更新通道，让自行构建的 APK 自动安装其发布版本。

调试阶段默认关闭自动更新，更新由当前工程明确控制。

如果需要联网下载构建依赖，应列出实际下载来源，避免不可复现或不可信的文件。

---

## 十一、开发输出和记录要求

你可以自行选择最适合的内部代码结构，但必须保留清晰的工程记录。

建议至少形成：

**源码版本记录**

列出五个项目实际采用的 Commit SHA、子模块和依赖版本。

**补丁记录**

每项修改对应一个明确的问题和来源，方便回退。

**构建记录**

保留 APK 版本、SHA256、包名、签名状态和构建结果。

**测试记录**

记录设备、构建版本、游戏、测试配置、帧率、错误及判断结果。

**问题清单**

明确哪些问题已经解决、仍然存在、尚未验证。

你可以使用 Markdown 文件完成这些工作，不要求开发复杂的管理系统、自动化平台或额外 UI。

所有项目文件应保存在当前明确指定的工程工作区，优先使用相对路径。

代码文件中的重要修改应有适当注释，但不要增加无意义的大段注释。

---

## 十二、验收标准

### 第一阶段验收：工程可复现

- 能成功从源码构建 APK。
- APK 具有独立包名。
- 不覆盖用户已有应用。
- 能通过签名校验。
- 有明确的 Commit 和构建日志。

### 第二阶段验收：RP4 Pro 基线

- ADB 能正确识别设备。
- 应用能够启动。
- Steam 运行环境能够启动。
- 游戏模式正确显示。
- 手柄、声音可用。
- 可以运行至少一款用户实际拥有的测试游戏。
- 无明显资源泄漏或异常崩溃。

### 第三阶段验收：整合成果

- 保留已验证的 G77 专项修复。
- 成功引入至少一项有实际价值的新补丁，或经分析证明当前版本已具备最优可用组合。
- 证明新增补丁没有破坏基线。
- 至少提供一组可复核的前后测试对比。

### 第四阶段验收：维护能力

- 能重复构建。
- 能升级 MaliBridge。
- 能识别官方 DroidDeck 后续可移植的补丁。
- 实验驱动不会影响稳定版本。
- 能方便回退。
- 不需要每次重新手工拼接 APK。

---

## 十三、工作执行方式

请不要只回复一份项目规划。

我希望你按照以上内容实际开展工程工作。

**开始时执行以下工作：**

1. 阅读五个项目的最新 GitHub 信息。
2. 确认当前工作目录及构建环境。
3. 检查目标设备是否已经通过 ADB 连接。
4. 识别当前安装的软件和设备参数，但不修改它们。
5. 确定 yujikaido `source` 基线的实际 Commit。
6. 分析 MaliBridge 子模块和关键补丁。
7. 建立源码版本及整合差异记录。
8. 尝试完成第一版基线 APK 编译。

然后根据编译和设备测试结果，继续推进后续阶段。

**工作纪律：**

- 不要反复向我询问已经提供过的信息。
- 不要进行无关的重构或优化。
- 不要擅自卸载已有应用。
- 不要主动申请不必要的系统权限。
- 不要擅自修改 Android 系统配置。
- 不要因为构建失败就降低项目安全要求。
- 每完成一个阶段，应明确说明已完成内容、真实测试结果和剩余问题。
- 区分已经验证、仅编译通过和理论可行。
- 如果遇到真正需要用户授权的危险操作，说明具体操作再请求同意。
- 如果某项技术不可行，应保留已经成功的部分，不要为了追求最终目标破坏现有成果。

最终目标是获得一个真正可以在 **Retroid Pocket 4 Pro / Mali-G77 MC9** 上持续使用、能够自行升级维护的专用 DroidDeck。

**请从实际源码检查和环境确认开始，不要假设你已经了解这些项目的实现细节，也不要只进行概念性讨论。**
