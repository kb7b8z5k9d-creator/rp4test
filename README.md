# RP4 Pro DroidDeck（自编译版）

给 **Retroid Pocket 4 Pro** 自己编译的 DroidDeck（在 Android 上跑 Steam / PC 游戏的运行时）。
非官方构建，自签名，独立包名，可以和官方版并存。

- 设备：Retroid Pocket 4 Pro（Mali-G77 MC9，Android 13，1334×750）
- 基线：[yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro) `source` 分支，已合并上游 fix46–fix51
- 图形桥：[ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge) v0.5.4.2（fix44 的 MSAA errata 手工并入）
- 源码仓库：[kb7b8z5k9d-creator/rp4develop](https://github.com/kb7b8z5k9d-creator/rp4develop)
- 包名：`com.rp4pro.droiddeck.lab`，签名 `CN=RP4Pro Lab`

## 这个版本做了什么

**PanVK 安装即用版（默认）**

- 内置 faux123 PanVK v1.1.3，支持导入、切换和直接加载，并自动检测 dma-buf 导出失败、使用兼容回退。已在 RP4 Pro 上验证能进入 Steam；ForestTD 使用 Vulkan / Forward Mobile 时可正常进入并游玩，截图中的测试场景约 50 FPS。该数据只代表当时场景。
- 小小梦魇增强版已在 RP4 Pro / PanVK / Proton Experimental (ARM64) 使用单独的通用 VKD3D 组件通过 D3D12 进入游戏，画面正常，但帧率很低，尚未优化。ForestTD 默认 D3D12 路径在此修复后尚未复测；此前 Vulkan 对照使用启动参数 `%command% --rendering-method mobile --rendering-driver vulkan`。
- PanVK 驱动（faux123 v1.1.3）和 VKD3D 组件已经打进 APK：首次启动自动安装，RP4 Pro 默认使用 PanVK；其他设备保持系统 Mali。VKD3D 组件是通用内存回退修复，对当前 Proton 的 64 位游戏生效；其他 Proton 版本和 32 位路径尚未验证。
- 当前版本 [v0.5.0](../../releases/tag/v0.5.0) 已内置 PanVK 和 VKD3D；此前的 [DX12 预览](../../releases/tag/v0.3.0-dx12-preview)、[PanVK 预览](../../releases/tag/v0.2.0-panvk-preview) 与稳定版 v0.1.1-fix51 仍保留。
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

## 图形模式（默认 PanVK，可一键切回）

安装后自动配置成 RP4 Pro 的默认体验：

- PanVK（Vulkan 1.3.363）加系统 Mali 桥接；
- Proton Experimental (ARM64) 自动启用 VKD3D 回退组件；
- Steam 使用 GPU 菜单，并关闭 Chromium 自己的 Vulkan 合成（修 Big Picture 闪烁）。

只有检测到 MT8797 / Retroid Pocket 4 才会自动启用；其他机型保持系统 Mali。

### 三种模式的区别

1. **系统 Mali + MaliBridge**：手机自带驱动，Vulkan 1.1.177。兼容性和帧率最好，D3D11/OpenGL 游戏优先用这个。
2. **PanVK**：内置的 faux123 v1.1.3，Vulkan 1.3.363，带 `VK_EXT_transform_feedback`、`geometryStreams=1` 和 AHB/dma-buf。适合需要 Vulkan 1.3 或 D3D12 的游戏；帧率通常低于 Mali。
3. **PanVK + VKD3D 回退组件**：在 PanVK 上跑 D3D12（VKD3D-Proton）时修复 READBACK 内存类型。只对当前 Proton Experimental (ARM64) 的 64 位路径验证过，其他 Proton 和 32 位未验证。

CEF 设置不是独立模式，它只影响 Steam 界面，和游戏渲染无关。

补充一点容易混淆的：**运行时驱动**是 Linux 侧的 Turnip，只在 Linux 自己加载 Vulkan 时用；**显示驱动**是 Android 侧的驱动，也是 Mali 桥接实际给游戏用的那个。RP4 上只需要切“图形模式”，不用动运行时驱动。

### 怎么切

组件页顶部现在直接有“图形模式”：启用 PanVK / 切回系统 Mali，点完重启会话。也可以从 配置 → 组件 → GPU 驱动 → 分别设置运行时驱动和显示驱动 → 显示驱动 进入同一个开关。

- 点“启用 PanVK” → 重启会话（退出 Steam 再进）；
- 点“切回系统 Mali” → 重启会话；
- 如果提示“请先导入 PanVK 驱动”，说明自动配置没跑或驱动被删除，可以从 Release 重新导入 `panvk_faux123_v1.1.3.adpkg.zip`。

组件管理器里可以单独恢复 Proton 自带的 VKD3D：Proton Experimental (ARM64) → VKD3D-Proton → 选择原始版本。

### 预期

- ForestTD 默认 D3D12：一次实测约 45 fps（含 Win-FG 2×；原生约 23 fps），能正常游玩，不代表所有场景；
- 小小梦魇增强版：能进入、画面正常，帧率低；
- Steam 界面：GPU 菜单约 60 fps，不闪；
- 需要帧率时切回系统 Mali；需要 D3D12 / Vulkan 1.3 时用 PanVK。

## 安装

1. 在 [Releases](../../releases) 下载 APK。
2. 允许安装未知来源应用，安装即可（包名独立，不会覆盖官方版）。
3. 首次启动按提示下载 Linux 运行时，然后登录 Steam。
4. 自签名构建：以后更新要覆盖安装（保持同一签名）。
5. 安装后默认就是 PanVK 配置：驱动和 VKD3D 组件已内置，首次启动自动安装（Proton Experimental 出现后自动启用）；想切回系统 Mali 或改回来见上面的“图形模式”。

v0.5.0 安装即用版 APK SHA-256：`b92ac32bf469a59ba1b02709cfb0b815b74d515a71bb3c1f121487ae33b177f1`（内置 PanVK v1.1.3 和 VKD3D 组件）

v0.3.0 DX12 预览版 APK SHA-256：`c1edd69d24ff4e531516a9e7e944ac0e3c3cdd8e8e760bbc7f0ed8a94abc677a`；VKD3D 组件 SHA-256：`3c3043bdb57f18a7bc8b93f36acf846e35693f7eb4c7b97f2fb7f5922dc8cfa6`。

稳定版 v0.1.1-fix51 APK SHA-256：`8c8c41079e6fcd4dd62b5fc9a5c67a0071834d79ef0c7ef440ab7c5ed79aa002`。

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

**Install-and-play PanVK build (default)**

- Bundles faux123's PanVK v1.1.3, imports, switches to and directly loads it, automatically probing for failed dma-buf export and enabling the existing fallback. On RP4 Pro, Steam started and ForestTD ran using Vulkan / Forward Mobile; the captured scene showed about 50 FPS, which is not a general benchmark.
- Little Nightmares Enhanced reached D3D12 gameplay with normal graphics on RP4 Pro / PanVK / Proton Experimental (ARM64), using the separately installed generic VKD3D component. Frame rate is very low and has not been optimized. ForestTD's default D3D12 path has not been retested with this fix; its earlier Vulkan control used `%command% --rendering-method mobile --rendering-driver vulkan`.
- The PanVK driver (faux123 v1.1.3) and the VKD3D component are bundled in the APK: the first start installs them, and an RP4 Pro uses PanVK by default; other devices stay on the system Mali driver. The component provides a general memory fallback for 64-bit games using the selected Proton; other Proton versions and the 32-bit path remain unverified.
- The current [v0.5.0](../../releases/tag/v0.5.0) bundles PanVK and VKD3D. The earlier [DX12 preview](../../releases/tag/v0.3.0-dx12-preview), [PanVK preview](../../releases/tag/v0.2.0-panvk-preview) and stable v0.1.1-fix51 remain available.
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

## Graphics modes (PanVK by default, one-tap switch back)

The first start configures the RP4 Pro default:

- PanVK (Vulkan 1.3.363) plus the system Mali bridge;
- the VKD3D fallback component selected for Proton Experimental (ARM64);
- GPU Steam menus, with Chromium's own Vulkan compositing off (the Big Picture flicker fix).

Only an MT8797 / Retroid Pocket 4 gets this profile; other devices keep the system Mali driver.

### The three modes

1. **System Mali + MaliBridge**: the phone's own driver, Vulkan 1.1.177. Best compatibility and frame rate; use it for D3D11 and OpenGL games.
2. **PanVK**: the bundled faux123 v1.1.3, Vulkan 1.3.363, with `VK_EXT_transform_feedback`, `geometryStreams=1` and AHB/dma-buf. For games that need Vulkan 1.3 or D3D12; the frame rate is usually lower than Mali.
3. **PanVK + VKD3D fallback**: fixes the READBACK memory type for D3D12 (VKD3D-Proton) on PanVK. Verified only for the 64-bit path of the current Proton Experimental (ARM64); other Protons and the 32-bit path are unverified.

The CEF setting is not a mode: it changes only the Steam UI, not game rendering.

One distinction that causes confusion: the **runtime driver** is the Linux-side Turnip, used only when Linux loads Vulkan itself; the **display driver** is the Android-side driver, and it is what the Mali bridge gives the games. On the RP4 only the "Graphics mode" switch matters; leave the runtime driver alone.

### Switching

The Components page now has the "Graphics mode" switch at the top: enable PanVK or switch back to Mali, then restart the session. The same switch is also under Settings → Components → GPU drivers → Set the runtime and display drivers separately → Display driver.

- "Enable PanVK" → restart the session (leave Steam and start it again);
- "Switch to Mali" → restart the session;
- If it says "Import a PanVK driver first", the automatic setup did not run or the driver was removed; import `panvk_faux123_v1.1.3.adpkg.zip` from the Release.

The component manager can restore Proton's own VKD3D separately: Proton Experimental (ARM64) → VKD3D-Proton → choose the original.

### What to expect

- ForestTD's default D3D12 path: about 45 fps in one measured session (with Win-FG 2×; about 23 fps native), playable; not a benchmark of every scene;
- Little Nightmares Enhanced: starts with normal graphics, low frame rate;
- Steam UI: GPU menus at about 60 fps, no flicker;
- Switch back to Mali when frame rate matters; use PanVK for D3D12 / Vulkan 1.3.

## Install

1. Download the APK from [Releases](../../releases).
2. Allow installing apps from unknown sources and install it (the package is independent, the
   official app is not replaced).
3. On first start, download the Linux runtime as prompted, then sign in to Steam.
4. Self-signed build: future updates must be installed over this one (same signature).
5. The PanVK profile is the default after installing: the driver and the VKD3D component are built in, installed on the first start (the component is selected once Proton Experimental exists). To go back to system Mali, see "Graphics modes" above.

v0.5.0 install-and-play APK SHA-256: `b92ac32bf469a59ba1b02709cfb0b815b74d515a71bb3c1f121487ae33b177f1` (PanVK v1.1.3 and the VKD3D component are built in)

v0.3.0 DX12 preview APK SHA-256: `c1edd69d24ff4e531516a9e7e944ac0e3c3cdd8e8e760bbc7f0ed8a94abc677a`; VKD3D component SHA-256: `3c3043bdb57f18a7bc8b93f36acf846e35693f7eb4c7b97f2fb7f5922dc8cfa6`.

Stable v0.1.1-fix51 APK SHA-256: `8c8c41079e6fcd4dd62b5fc9a5c67a0071834d79ef0c7ef440ab7c5ed79aa002`.

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
