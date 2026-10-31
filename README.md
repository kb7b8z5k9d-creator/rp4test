# RP4 Pro DroidDeck · v0.5.1 玩家版

给 **Retroid Pocket 4 Pro** 编译的 DroidDeck，可在 Android 上运行 Steam / PC 游戏。非官方、自签名构建，包名 `com.rp4pro.droiddeck.lab`，可以与官方版并存。

**首次使用必须检查两项：手动开启 Mali Bridge，并手动下载、安装 Linux 运行时（支持包）。** PanVK 和 VKD3D 已内置，无需另找驱动 ZIP 或替换游戏 DLL。

[下载 v0.5.1 APK](https://github.com/kb7b8z5k9d-creator/rp4test/releases/download/v0.5.1/RP4ProLab-v0.5.1.apk) · [版本说明](https://github.com/kb7b8z5k9d-creator/rp4test/releases/tag/v0.5.1) · [完整源码](https://github.com/kb7b8z5k9d-creator/rp4develop/tree/v0.5.1)

## 新手首次安装

1. **安装 APK。** 下载上方安装包，在 Android 提示时允许安装未知来源应用，然后打开 **RP4Pro Lab**。
2. **开启 Bridge。** 打开 **配置 → 概览**，找到显卡区域。若显示 **“Mali 桥接：关闭”**，点击使其变成 **“Mali 桥接：开启”**；已经开启则不用再点。RP4 Pro 需要图形桥接，选择 PanVK 不能代替开启 Bridge。若按钮不可点击，先结束正在运行的 Steam 会话。
3. **安装 Linux 支持包。** 在同一页找到 **“Linux 运行时”**，点击 **“安装”**，保持联网，等待下载和解压完成。这就是所需的 Linux 支持环境；玩 Steam 不需要另装“Linux 桌面”。APK 内置驱动不代表包含完整 Linux 运行时。
4. **启动 Steam。** 首次会下载所需的 **Proton Experimental (ARM64)**，随后自动准备 VKD3D。等“正在准备游戏组件”完成后，用自己的 Steam 账号登录；不用为了安装 VKD3D 再退出、重开应用。
5. **检查图形模式。** 在 **配置 → 组件** 顶部查看“图形模式”。PanVK 已内置；若当前为系统 Mali，而你要运行需要 Vulkan 1.3 / D3D12 的游戏，点“启用 PanVK”，然后重启 Steam 会话。更新安装会保留已有驱动选择。
6. **安装并运行游戏。** 帧生成默认关闭，需要时自行开启。

后续更新通常直接覆盖安装，保持相同包名和签名。**v0.5.1 仍需手动完成上述第 2、3 步，请不要跳过。**

## v0.5.1 改进

- **首次安装顺序**：Proton 安装完成后，先准备并核验 VKD3D，再启动 Steam，无需等待下次启动应用。
- **恢复与选择保护**：内置资源校验后原子安装；Proton 更新或重装后重新核对。保留已有驱动、菜单和组件选择，主动恢复原版 VKD3D 后不会被自动覆盖。
- **玩家路径清理**：移除启动 smoke / vulkaninfo 测试、旧诊断参数文件入口和默认逐秒帧统计写盘。Release APK 不可调试，不打包 smoke 程序；保留必要的驱动兼容性检测、自动回退和正常错误日志。
- **沿用实测组件**：内置 faux123 PanVK v1.1.3 和通用 VKD3D readback 回退，按实际驱动能力选择 DXVK。
- 本机用户已确认本版可正常使用；电脑侧 **45 项回归检查通过**。本次没有进行新的性能优化或帧率对照测试。

## 图形模式与验证范围

RP4 Pro 的系统 Mali 驱动报告 Vulkan 1.1.177；内置 PanVK 报告 Vulkan 1.3.363。需要 Vulkan 1.3 / D3D12 的游戏可使用 PanVK。组件页顶部可以切回系统 Mali，切换后重启 Steam 会话。

Bridge 开关和 PanVK 驱动选择是两项设置，RP4 Pro 使用 PanVK 时也需要开启 Bridge。Linux 侧的运行时驱动通常无需调整。Steam 保留 GPU 菜单及 CEF 防闪烁配置；帧生成是独立选项。

VKD3D 的通用修复允许缺少 HOST_CACHED 时回退到真实可用的 HOST_VISIBLE 内存，不针对某个游戏。已验证范围为 **Proton Experimental (ARM64) 的 64 位路径**；其他 Proton、32 位路径和其他机型尚未完整验证。

## 游戏与已知问题

- **小小梦魇增强版**：此前已通过 D3D12 进入游戏，画面正常、无闪烁，但帧率低。
- **ForestTD**：此前已验证默认 D3D12 可玩。一次测试约 45 fps，含 Win-FG 2×，原生约 23 fps；更早的 Vulkan / Forward Mobile 场景曾约 50 fps。不同渲染方式、场景和设置的数据不能直接比较驱动性能，也不代表 v0.5.1 的新基准。
- **Prison Architect**：此前系统 Mali / zink 路径的地面贴图闪烁尚未解决。
- **Crimsonland**：中文文字偏移或重影尚未定位。

若首次启动提示 GPU 不受支持或无法进入会话，先检查 **配置 → 概览** 中 Bridge 已开启、Linux 运行时已安装。需要反馈时可使用应用内日志分享；安装不需要 ADB 调试。

## 下载校验

- APK：`RP4ProLab-v0.5.1.apk`，71,842,487 字节（约 68.51 MiB）。
- SHA-256：`597cb6a1d8c65d978c3bf6ce0f0746db0e3b0c50772508ee39b95e71c0938890`。
- 版本 0.5.1 / versionCode 15；包名 `com.rp4pro.droiddeck.lab`；签名 `CN=RP4Pro Lab`，与旧版一致。

发布附件是本机用户实测的同一份 APK。[历史版本](https://github.com/kb7b8z5k9d-creator/rp4test/releases)保留。

## 源码与构建

- 完整源码：[rp4develop · v0.5.1](https://github.com/kb7b8z5k9d-creator/rp4develop/tree/v0.5.1)，`source` / `driver-import` 同步至 `56c4bd9b60fe0b29d54d70d0e50168a2a9447927`。
- 本次增量：[patches/v0.5.1-player.patch](patches/v0.5.1-player.patch)，相对 `d37319fd`；此前内置的驱动和 WCP 资源在完整源码中。
- 历史 `fix51-local-changes.patch`、`driver-import.patch` 对应旧基线，复现当前版本请使用完整源码标签。
- 基线：[yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro) fix51（含 fix46–fix51）；MaliBridge v0.5.4.2。
- Windows + WSL2 Ubuntu 24.04 + Docker，NDK 27.3.13750724、JDK 17；构建入口 `tools/build_local.sh`，玩家 APK 使用 Release 构建。私钥不在仓库中。

## 上游项目

- [Droid-Deck/DroidDeck](https://github.com/Droid-Deck/DroidDeck)
- [ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge)
- [yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro)
- [The412Banner/droiddeck-nonadreno-drivers](https://github.com/The412Banner/droiddeck-nonadreno-drivers)
- [faux123/panvk-faux123-driver-rp4pro](https://github.com/faux123/panvk-faux123-driver-rp4pro)

本构建非上述项目官方发布，不包含 Steam 凭据或设备标识。

---

# RP4 Pro DroidDeck · v0.5.1 player build

An unofficial, self-signed DroidDeck build for the **Retroid Pocket 4 Pro**, running Steam / PC games on Android. Package `com.rp4pro.droiddeck.lab` can coexist with the official app.

**First-time users must enable the Mali bridge and manually download/install the Linux runtime support package.** PanVK and VKD3D are bundled; no separate driver ZIP or game-local DLL replacement is needed.

[Download v0.5.1 APK](https://github.com/kb7b8z5k9d-creator/rp4test/releases/download/v0.5.1/RP4ProLab-v0.5.1.apk) · [Release notes](https://github.com/kb7b8z5k9d-creator/rp4test/releases/tag/v0.5.1) · [Full source](https://github.com/kb7b8z5k9d-creator/rp4develop/tree/v0.5.1)

## First-time setup

1. **Install the APK.** Allow installation from unknown sources when Android asks, then open **RP4Pro Lab**.
2. **Enable the bridge.** Open **Setup → Overview**, find the GPU section, and change **“Mali bridge: off”** to **“Mali bridge: on”**. Leave it on if already enabled. Choosing PanVK does not replace enabling the bridge. If the button is disabled, end the running Steam session first.
3. **Install Linux support.** On the same page, find **“Linux runtime”**, tap **“Install”**, and wait for download and extraction. This is the required support environment; Steam does not require the separate Linux desktop package. Bundling drivers does not bundle the full Linux runtime.
4. **Start Steam.** The first launch downloads **Proton Experimental (ARM64)** as needed, then prepares VKD3D. Wait for “Preparing game components” and sign in with your own Steam account. No extra app restart is required to install VKD3D.
5. **Check graphics mode.** At the top of **Setup → Components**, check “Graphics mode”. PanVK is bundled. If system Mali is selected and your game needs Vulkan 1.3 / D3D12, enable PanVK and restart the Steam session. Updates preserve existing driver choices.
6. **Install and play games.** Frame generation is off by default; enable it yourself if desired.

Future updates normally install over the existing app with the same package and signature. **v0.5.1 still requires the manual setup in steps 2–3.**

## Changes in v0.5.1

- Prepare and verify VKD3D after Proton installation and before Steam starts, without waiting for another app launch.
- Verify assets and install atomically; recheck after Proton updates/reinstalls. Preserve player choices; explicitly restoring original VKD3D opts out of the automatic default.
- Remove startup smoke / vulkaninfo tests, legacy diagnostic-file overrides and default per-second frame-statistics writes. The release APK is not debuggable and excludes the smoke binary. Required compatibility probes, automatic fallback and normal error logs remain.
- Retain the validated faux123 PanVK v1.1.3 and generic VKD3D readback fallback; select DXVK from actual driver capabilities.
- The RP4 Pro owner confirmed this version works; **45 computer-side regression checks passed**. No new performance optimization or controlled frame-rate comparison was performed.

## Graphics and validation scope

The stock Mali driver reports Vulkan 1.1.177; bundled PanVK reports Vulkan 1.3.363. PanVK can be used for games requiring Vulkan 1.3 / D3D12. Switch back to system Mali at the top of the Components page, then restart the Steam session.

The bridge switch and PanVK selection are separate settings; PanVK on RP4 Pro also needs the bridge. The Linux-side runtime driver normally needs no adjustment. Steam retains GPU menus and its CEF flicker fix; frame generation is separate.

The generic VKD3D fix falls back to a real HOST_VISIBLE memory type when HOST_CACHED is unavailable, without game-specific patches. The validated scope is the **64-bit path of Proton Experimental (ARM64)**; other Protons, 32-bit paths and other devices are not fully validated.

## Games and known issues

- **Little Nightmares Enhanced:** previously reached D3D12 gameplay with normal graphics and no flicker, but low frame rate.
- **ForestTD:** previously playable with default D3D12. One session measured about 45 fps with Win-FG 2×, about 23 fps native; an earlier Vulkan / Forward Mobile scene showed about 50 fps. Different renderers, scenes and settings make these unsuitable for comparing driver performance or as a new v0.5.1 benchmark.
- **Prison Architect:** previously observed stock-Mali / zink ground-texture flicker is unresolved.
- **Crimsonland:** Chinese text offset or ghosting is unresolved.

If the first launch reports an unsupported GPU or cannot start a session, check **Setup → Overview** for an enabled bridge and an installed Linux runtime first. Use the app's log-sharing feature for reports; installation does not require ADB debugging.

## Download verification

- APK: `RP4ProLab-v0.5.1.apk`, 71,842,487 bytes (about 68.51 MiB).
- SHA-256: `597cb6a1d8c65d978c3bf6ce0f0746db0e3b0c50772508ee39b95e71c0938890`.
- Version 0.5.1 / versionCode 15; package `com.rp4pro.droiddeck.lab`; signer `CN=RP4Pro Lab`, unchanged from earlier builds.

The release attachment is the same APK validated by the RP4 Pro owner. [Previous versions](https://github.com/kb7b8z5k9d-creator/rp4test/releases) remain available.

## Source and build

- Full source: [rp4develop · v0.5.1](https://github.com/kb7b8z5k9d-creator/rp4develop/tree/v0.5.1); `source` / `driver-import` are synchronized at `56c4bd9b60fe0b29d54d70d0e50168a2a9447927`.
- Incremental patch: [patches/v0.5.1-player.patch](patches/v0.5.1-player.patch), relative to `d37319fd`. Previously bundled driver/WCP assets are in the full source tree.
- `fix51-local-changes.patch` and `driver-import.patch` are historical material for their original baselines; use the source tag to reproduce this version.
- Base: [yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro) fix51, including fix46–fix51; MaliBridge v0.5.4.2.
- Windows + WSL2 Ubuntu 24.04 + Docker, NDK 27.3.13750724, JDK 17; entry point `tools/build_local.sh`. Use a Release build for the player APK. Private signing keys are not in the repository.

## Upstream projects

- [Droid-Deck/DroidDeck](https://github.com/Droid-Deck/DroidDeck)
- [ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge)
- [yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro)
- [The412Banner/droiddeck-nonadreno-drivers](https://github.com/The412Banner/droiddeck-nonadreno-drivers)
- [faux123/panvk-faux123-driver-rp4pro](https://github.com/faux123/panvk-faux123-driver-rp4pro)

This build is unofficial and contains no Steam credentials or device identifiers.
