# RP4 Pro DroidDeck（自编译版）

给 **Retroid Pocket 4 Pro** 自己编译的 DroidDeck（在 Android 上跑 Steam / PC 游戏的运行时）。
非官方构建，自签名，独立包名，可以和官方版并存。

- 设备：Retroid Pocket 4 Pro（Mali-G77 MC9，Android 13，1334×750）
- 基线：[yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro) `source` 分支，已合并上游 fix46–fix51
- 图形桥：[ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge) v0.5.4.2（fix44 的 MSAA errata 手工并入）
- 包名：`com.rp4pro.droiddeck.lab`，签名 `CN=RP4Pro Lab`

## 这个版本做了什么

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

APK 校验（当前里程碑 v0.1.1-fix51）：`8c8c41079e6fcd4dd62b5fc9a5c67a0071834d79ef0c7ef440ab7c5ed79aa002`

## 构建

Windows + WSL2 Ubuntu 24.04 + Docker（交叉编译镜像 `droiddeck-local-cross:24.04-v3`），
NDK 27.3.13750724、JDK 17。构建脚本和步骤见 `docs/整合方案.md` 与项目内的 `tools/build_local.sh`。

本地改动以补丁形式附在本仓库：`patches/fix51-local-changes.patch`（相对 fix51 基线，24 个文件），
可用 `git apply` 应用到 `yujikaido/droiddeck-rp4pro` 的 fix51 源码；签名密钥不在仓库里，需自行生成。

## 上游项目

- [Droid-Deck/DroidDeck](https://github.com/Droid-Deck/DroidDeck)
- [ASEbrahim/MaliBridge](https://github.com/ASEbrahim/MaliBridge)
- [yujikaido/droiddeck-rp4pro](https://github.com/yujikaido/droiddeck-rp4pro)
- [The412Banner/droiddeck-nonadreno-drivers](https://github.com/The412Banner/droiddeck-nonadreno-drivers)
- [faux123/panvk-faux123-driver-rp4pro](https://github.com/faux123/panvk-faux123-driver-rp4pro)（PanVK，未实测）

## 说明

这是个人测试构建，不是 DroidDeck 或 MaliBridge 的官方发布。使用风险自负；
不包含任何 Steam 凭据或设备标识。测试报告见 `docs/RP4Pro-test-report.md`。
