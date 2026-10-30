# RP4 Pro DroidDeck（自编译版）

给 **Retroid Pocket 4 Pro** 自己编译的 DroidDeck（在 Android 上跑 Steam / PC 游戏的运行时）。
非官方构建，自签名，独立包名，可以和官方版并存。

- 设备：Retroid Pocket 4 Pro（Mali-G77 MC9，Android 13，1334×750）
- 基线：[yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro) `source` 分支，已合并上游 fix46–fix51
- 图形桥：[ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge) v0.5.4.2（fix44 的 MSAA errata 手工并入）
- 源码仓库：[kb7b8z5k9d-creator/rp4develop](https://github.com/kb7b8z5k9d-creator/rp4develop)
- 包名：`com.rp4pro.droiddeck.lab`，签名 `CN=RP4Pro Lab`

## 这个版本做了什么

**PanVK 实验版**

- 支持导入、切换和直接加载 PanVK，并自动检测 dma-buf 导出失败、使用兼容回退。已在 RP4 Pro 上验证能进入 Steam；ForestTD 使用 Vulkan / Forward Mobile 时可正常进入并游玩，截图中的测试场景约 50 FPS。该数据只代表当时场景。
- 小小梦魇增强版已在 RP4 Pro / PanVK / Proton Experimental (ARM64) 使用单独的通用 VKD3D 组件通过 D3D12 进入游戏，画面正常，但帧率很低，尚未优化。ForestTD 默认 D3D12 路径在此修复后尚未复测；此前 Vulkan 对照使用启动参数 `%command% --rendering-method mobile --rendering-driver vulkan`。
- PanVK 驱动和 VKD3D 组件均不包含在 APK 中，需要分别导入。VKD3D 组件是通用内存回退修复，对当前 Proton 的 64 位游戏生效；其他 Proton 版本和 32 位路径尚未验证。
- [DX12 里程碑预发布](../../releases/tag/v0.3.0-dx12-preview) 提供 APK 和 VKD3D 组件；此前 [PanVK 预发布](../../releases/tag/v0.2.0-panvk-preview) 与稳定版 v0.1.1-fix51 仍保留。
- 源码 `source` / `driver-import` 的本次版本：`df822f1f9a48f2dc54af3ddeb1afe86a626b45af`。

**修复**

- **Steam Big Picture 闪烁**：来自 Chromium 自己的 Vulkan 合成。会话环境里设置
  `BL_VKBRIDGE_CEF_VULKAN=0`（ANGLE 仍走 Vulkan）后消失，对所有 Steam 页面和游戏生效。
- **整画面闪烁（ForestTD 等）**：两个门槛都修了。游戏窗口可能是 1280×722，也可能 1152×648（源矩形
  1151×647），合成器原来要求“窗口严格等于场景、源矩形严格等于整图”才给独立显示层，差一两个像素就
  整场退回复制路径并闪烁。现在允许窗口 ±16 像素、源矩形 ±2 像素，并按窗口的实际位置和大小映射到
  显示层；64×64 以下的触摸光标窗口也不再算“压在游戏上面”。
- **上游补丁**：已合并 fix46–fix51，包括 MaliBridge v0.5.4.2、fix44 `msaa2_as_4`
  （Mali-G77 只有 1×/4× 采样，D3D9 游戏要 2× 会退出，改为按 4× 创建）、抗锯齿开关、最快核心优先、
  退 Retro 游戏时关闭模拟器等；DroidDeck 官方 0.6.7 之后的修复（GTK3、游戏目录识别、Steam UI 刷新率）
  也都在里面。
- **性能补丁**：DroidDeck 官方的 CPU 拓扑、proot fast path、compositor tick/render-on-commit。
  在这台设备上均确认生效，帧率无明显变化，按“无负面影响”保留。

**已知问题**

- Prison Architect 的地面贴图仍会闪。它走 OpenGL → Mesa zink → Vulkan → Mali 这条链，
  与显示层无关；强制同步、关 GL 线程、游戏兼容模式都试过，无效，怀疑在 zink 的纹理/mipmap 路径。
- Crimsonland 的中文文字有轻微偏移/重影。不是缩放采样、不是 DXVK、不是系统字体替换
  （字体打包在游戏资源里），原因仍未定位。

## 安装

1. 在 [Releases](../../releases) 下载 APK。
2. 允许安装未知来源应用，安装即可（包名独立，不会覆盖官方版）。
3. 首次启动按提示下载 Linux 运行时，然后登录 Steam。
4. 自签名构建：以后更新要覆盖安装（保持同一签名）。
5. 使用 DX12 里程碑时，另从 Release 下载 `vkd3d-3.1-readback-fallback.wcp`，在应用的组件管理器中导入并为 Proton Experimental (ARM64) 选择它；管理器会保存原组件用于回退。另行导入并启用 PanVK 驱动。

APK 校验（当前里程碑 v0.1.1-fix51）：`8c8c41079e6fcd4dd62b5fc9a5c67a0071834d79ef0c7ef440ab7c5ed79aa002`

PanVK 实验 APK SHA-256：`2708b362dd27fd39e3d6a5d24a2eef6f5795e5efb32854923ac14ae660411160`

DX12 里程碑 APK SHA-256：`c1edd69d24ff4e531516a9e7e944ac0e3c3cdd8e8e760bbc7f0ed8a94abc677a`；VKD3D 组件 SHA-256：`3c3043bdb57f18a7bc8b93f36acf846e35693f7eb4c7b97f2fb7f5922dc8cfa6`。

## 构建

Windows + WSL2 Ubuntu 24.04 + Docker（交叉编译镜像 `droiddeck-local-cross:24.04-v3`），
NDK 27.3.13750724、JDK 17。源码与构建说明在源码仓库
[rp4develop](https://github.com/kb7b8z5k9d-creator/rp4develop)，构建入口是 `tools/build_local.sh`。

如果只想在官方 fix51 源码上复现改动：`patches/fix51-local-changes.patch`（相对 fix51 基线，24 个文件），
用 `git apply` 应用；签名密钥不在仓库里，需自行生成。

PanVK 接入补丁：`patches/driver-import.patch`，相对源码仓库 `af27c28c`；包含直接加载、自动导出回退、驱动能力识别和 VKD3D 源码补丁。VKD3D 组件的构建说明见源码仓库 `tools/vkd3d/`。

## 上游项目

- [Droid-Deck/DroidDeck](https://github.com/Droid-Deck/DroidDeck)
- [ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge)
- [yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro)
- [The412Banner/droiddeck-nonadreno-drivers](https://github.com/The412Banner/droiddeck-nonadreno-drivers)
- [faux123/panvk-faux123-driver-rp4pro](https://github.com/faux123/panvk-faux123-driver-rp4pro)（另行导入的 PanVK 驱动）

## 说明

这是个人测试构建，不是 DroidDeck 或 MaliBridge 的官方发布。使用风险自负；
不包含任何 Steam 凭据或设备标识。

---

# RP4 Pro DroidDeck (self-built)

A self-compiled DroidDeck build for the **Retroid Pocket 4 Pro** (the Android runtime that runs
Steam and PC games). Unofficial, self-signed, with its own package name, so it can live beside the
official app.

- Device: Retroid Pocket 4 Pro (Mali-G77 MC9, Android 13, 1334×750)
- Base: [yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro) `source`, with
  upstream fix46–fix51 merged
- Graphics bridge: [ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge) v0.5.4.2 (fix44's
  MSAA erratum hand-merged)
- Source repository: [kb7b8z5k9d-creator/rp4develop](https://github.com/kb7b8z5k9d-creator/rp4develop)
- Package: `com.rp4pro.droiddeck.lab`, signature `CN=RP4Pro Lab`

## What this build changes

**Experimental PanVK build**

- Imports, switches to and directly loads PanVK, automatically probing for failed dma-buf export and enabling the existing fallback. On RP4 Pro, Steam started and ForestTD ran using Vulkan / Forward Mobile; the captured scene showed about 50 FPS, which is not a general benchmark.
- Little Nightmares Enhanced reached D3D12 gameplay with normal graphics on RP4 Pro / PanVK / Proton Experimental (ARM64), using the separately installed generic VKD3D component. Frame rate is very low and has not been optimized. ForestTD's default D3D12 path has not been retested with this fix; its earlier Vulkan control used `%command% --rendering-method mobile --rendering-driver vulkan`.
- Neither the PanVK driver nor the VKD3D component is bundled in the APK; import them separately. The component provides a general memory fallback for 64-bit games using the selected Proton; other Proton versions and the 32-bit path remain unverified.
- The [DX12 milestone prerelease](../../releases/tag/v0.3.0-dx12-preview) provides an APK and VKD3D component. The earlier [PanVK prerelease](../../releases/tag/v0.2.0-panvk-preview) and stable v0.1.1-fix51 remain available.
- Source version on `source` / `driver-import`: `df822f1f9a48f2dc54af3ddeb1afe86a626b45af`.

**Fixes**

- **Steam Big Picture flicker**: it came from Chromium's own Vulkan compositing. Setting
  `BL_VKBRIDGE_CEF_VULKAN=0` in the session environment (ANGLE stays on Vulkan) removes it for every
  Steam screen and every game.
- **Whole-picture flicker (ForestTD and similar)**: two thresholds were fixed. A game window can be
  1280×722, or 1152×648 with a 1151×647 source rectangle; the compositor used to give a window its
  own display layer only when the window matched the scene exactly and the source matched the whole
  image, so a one-pixel difference dropped the whole scene to the copy path and flickered. It now
  accepts windows within ±16 px and sources within ±2 px, maps the buffer onto the window's actual
  rectangle, and no longer counts touch-cursor windows of 64×64 or less as "above the game".
- **Upstream patches**: fix46–fix51 are merged, including MaliBridge v0.5.4.2, fix44's `msaa2_as_4`
  (Mali-G77 multisamples at 1×/4× only; D3D9 games that ask for 2× used to quit, now the texture is
  made with 4 samples), the anti-aliasing switch, fastest-core-first ordering, closing Retro
  emulators on exit, and DroidDeck's post-0.6.7 fixes (GTK3, game-folder listing, Steam UI refresh
  pacing).
- **Performance patches**: DroidDeck's CPU topology, proot fast path and compositor
  tick/render-on-commit are included. All are confirmed active on this device; no measurable frame
  rate change, kept because they cause no regression.

**Known issues**

- Prison Architect's ground textures still flicker. They go through OpenGL → Mesa zink → Vulkan →
  Mali, independent of the display layer; forced synchronisation, disabling GL threading and the
  game's compatibility mode were all tried without effect. Suspected in zink's texture/mipmap path.
- Crimsonland's Chinese text sits slightly off or ghosts. It is not scaling, not DXVK and not system
  font substitution (the fonts are packed inside the game), and the cause is still unknown.

## Install

1. Download the APK from [Releases](../../releases).
2. Allow installing apps from unknown sources and install it (the package is independent, the
   official app is not replaced).
3. On first start, download the Linux runtime as prompted, then sign in to Steam.
4. Self-signed build: future updates must be installed over this one (same signature).
5. For the DX12 milestone, also download `vkd3d-3.1-readback-fallback.wcp` from the Release, import it in the component manager and select it for Proton Experimental (ARM64). The manager saves the original component for rollback. Obtain and enable PanVK separately.

APK checksum (current milestone v0.1.1-fix51): `8c8c41079e6fcd4dd62b5fc9a5c67a0071834d79ef0c7ef440ab7c5ed79aa002`

PanVK experimental APK SHA-256: `2708b362dd27fd39e3d6a5d24a2eef6f5795e5efb32854923ac14ae660411160`

DX12 milestone APK SHA-256: `c1edd69d24ff4e531516a9e7e944ac0e3c3cdd8e8e760bbc7f0ed8a94abc677a`; VKD3D component SHA-256: `3c3043bdb57f18a7bc8b93f36acf846e35693f7eb4c7b97f2fb7f5922dc8cfa6`.

## Build

Windows + WSL2 Ubuntu 24.04 + Docker (cross-compile image `droiddeck-local-cross:24.04-v3`),
NDK 27.3.13750724, JDK 17. The source and build notes are in the source repository
[rp4develop](https://github.com/kb7b8z5k9d-creator/rp4develop); the entry point is
`tools/build_local.sh`.

To reproduce the changes on the official fix51 tree instead:
`patches/fix51-local-changes.patch` (24 files against fix51), apply with `git apply`. The signing
key is not in the repository; generate your own.

The PanVK integration patch `patches/driver-import.patch` is relative to source commit `af27c28c` and includes direct loading, automatic export fallback, driver capability detection and the VKD3D source patch. See `tools/vkd3d/` in the source repository for component build notes.

## Upstream projects

- [Droid-Deck/DroidDeck](https://github.com/Droid-Deck/DroidDeck)
- [ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge)
- [yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro)
- [The412Banner/droiddeck-nonadreno-drivers](https://github.com/The412Banner/droiddeck-nonadreno-drivers)
- [faux123/panvk-faux123-driver-rp4pro](https://github.com/faux123/panvk-faux123-driver-rp4pro) (PanVK driver, imported separately)

## Notes

This is a personal test build, not an official DroidDeck or MaliBridge release. Use at your own
risk; it contains no Steam credentials or device identifiers.
