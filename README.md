# RP4 Pro DroidDeck（自编译版）

给 **Retroid Pocket 4 Pro** 自己编译的 DroidDeck（在 Android 上跑 Steam / PC 游戏的运行时）。
非官方构建，自签名，独立包名，可以和官方版并存。

- 设备：Retroid Pocket 4 Pro（Mali-G77 MC9，Android 13，1334×750）
- 基线：[yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro) `source` 分支（fix43），
  已合并上游 fix44 / fix45
- 图形桥：[ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge) v0.5.4.2（fix44 的 MSAA errata 手工并入）
- 包名：`com.rp4pro.droiddeck.lab`，签名 `CN=RP4Pro Lab`

## 这个版本做了什么

**修复**

- **Steam Big Picture 闪烁**：来自 Chromium 自己的 Vulkan 合成。会话环境里设置
  `BL_VKBRIDGE_CEF_VULKAN=0`（ANGLE 仍走 Vulkan）后消失，对所有 Steam 页面和游戏生效。
- **整画面闪烁（ForestTD 等）**：游戏全屏窗口是 1280×722，场景是 1280×720，原来要求严格相等才给
  独立显示层，差两像素就整场退回复制路径并闪烁。现在允许 ±16 像素，并忽略 64×64 以下的触摸光标窗口，
  触摸不再把游戏层踢掉。
- **上游补丁**：MaliBridge v0.5.4.2；fix44 `msaa2_as_4`（Mali-G77 只有 1×/4× 采样，
  D3D9 游戏要 2× 会退出，改为按 4× 创建）；fix45 已知游戏表（Ys: The Oath in Felghana 用 Wine 内置 d3dx9）。
- **性能补丁**：DroidDeck 官方的 CPU 拓扑、proot fast path、compositor tick/render-on-commit。
  在这台设备上均确认生效，帧率无明显变化，按“无负面影响”保留。

**已知问题**

- Prison Architect 的地面贴图仍会闪。它走 OpenGL → Mesa zink → Vulkan → Mali 这条链，
  与显示层无关；强制同步、关 GL 线程、游戏兼容模式都试过，无效，怀疑在 zink 的纹理/mipmap 路径。
- Crimsonland 的中文文字有轻微偏移/重影。不是缩放采样、不是 DXVK、不是系统字体替换
  （字体打包在游戏资源里），原因仍未定位。
- ForestTD 在默认 1152×648 窗口下仍可能闪；“按窗口实际矩形映射”的通用修复还在做。

## 安装

1. 在 [Releases](../../releases) 下载 APK。
2. 允许安装未知来源应用，安装即可（包名独立，不会覆盖官方版）。
3. 首次启动按提示下载 Linux 运行时，然后登录 Steam。
4. 自签名构建：以后更新要覆盖安装（保持同一签名）。

APK 校验（当前里程碑）：`59a43707985bbb0358845bfd71179fe2d306563e81be77519dfc5daa56af3beb`

## 构建

Windows + WSL2 Ubuntu 24.04 + Docker（交叉编译镜像 `droiddeck-local-cross:24.04-v3`），
NDK 27.3.13750724、JDK 17。构建脚本和步骤见 `docs/整合方案.md` 与项目内的 `tools/build_local.sh`。

## 上游项目

- [Droid-Deck/DroidDeck](https://github.com/Droid-Deck/DroidDeck)
- [ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge)
- [yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro)
- [The412Banner/droiddeck-nonadreno-drivers](https://github.com/The412Banner/droiddeck-nonadreno-drivers)
- [faux123/panvk-faux123-driver-rp4pro](https://github.com/faux123/panvk-faux123-driver-rp4pro)（PanVK，未实测）

## 说明

这是个人测试构建，不是 DroidDeck 或 MaliBridge 的官方发布。使用风险自负；
不包含任何 Steam 凭据或设备标识。测试报告见 `docs/RP4Pro-test-report.md`。
