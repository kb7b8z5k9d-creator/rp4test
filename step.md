# RP4 Pro DroidDeck — 项目进度（step.md）

最后更新：2026-10-29 18:56（东八区）
用途：额度耗尽、上下文压缩或换模型接手时，先读这份文件恢复现场。

---

## 0. 一句话现状

基线 B 已通过实机验收：Steam、Prison Architect、ForestTD 可运行，手柄与退出重进正常。当前进入 C1（MaliBridge v0.5.4.2 增量对比）。

---

## 1. 项目目标与方案

- 目标：给 Retroid Pocket 4 Pro（Mali-G77 MC9 / Android 13 / 原厂 Vulkan 1.1.177）做一套自己编译、自己签名、独立包名的 DroidDeck 定制版。
- 方案（1+3+1）：
  - 基线 B：yujikaido `droiddeck-rp4pro` 的 `source` 分支，锁定 `684939c59f9b488628f2527eaff6150a6d4c20a8`（fix43 发布所用源码）。
  - 零部件 C1：MaliBridge v0.5.4.2 相对 v0.5.4.1 的增量（三方合并，不整包覆盖）。
  - 零部件 C2：DroidDeck 官方 main 的通用修复（按功能域分批）。
  - 零部件 C3：The412Banner 的 Gamescope 组件（预编译二进制替换，A/B 对比）。
  - 实验 E：faux123 PanVK v1.1.2，独立分支、独立包名，不并入稳定线。
- 详细方案见 `docs/整合方案.md`；上游核对结论见 `docs/项目现状.md`。

---

## 2. 仓库与本地目录

- GitHub（公开）：`https://github.com/kb7b8z5k9d-creator/rp4test`
  - 目前只放文档：`docs/gpt交接词.md`、`docs/项目现状.md`、`docs/整合方案.md`、`.gitignore`
  - `temp/` 被 `.gitignore` 排除，不会上传
- 本地：`C:\Users\28157\Desktop\rp4`
  - `docs/`：正式文档
  - `src/`：yujikaido 源码（独立 Git 仓库，HEAD `684939c5`，`source` 分支）
  - `temp/`：历史材料、调查数据、构建脚本、日志、令牌（不入仓库）

---

## 3. 环境（已就绪）

- Windows 11，WSL2 + Ubuntu 24.04，用户名 `rp4test`
- Docker 29.1.3（WSL 内安装，服务已启动）
- WSL 内 Android SDK：`~/android-sdk`
  - platform-tools、platforms;android-34、build-tools;35.0.0、NDK 27.3.13750724、cmake 3.22.1
- JDK 17、zstd 1.5.5、gh 2.45.0、Python 3.12、make、patch、unzip
- 设备：Retroid Pocket 4 Pro，序列号 `JY2024…536`，ADB 可用（Windows SDK 的 adb.exe）
- GitHub 只读令牌：`temp/gitacc.txt`（5000 次/小时，勿提交）

---

## 4. 已完成

### 4.1 上游核对（2026-10-29）

- DroidDeck 官方 main：`d1bd23df161d`，最新 Release 0.6.0
- MaliBridge main：`95a1e600a512`，最新 Release v0.5.4.2
- PR #35（MaliBridge）：**open、未合并**，含 `nan_position`（MK-45）、mutable-format 查询重试、AHB twin 等
- DroidDeck PR #1692/#1693：**closed、未合并**
- yujikaido：main/fix43 `403fe974`，source `684939c5`（已内嵌 MaliBridge 472 文件，版本 v0.5.4.1 + G77 补丁）
- The412Banner：现在只发 gamescope/venus 组件（tzst），不发 APK
- faux123：v1.1.2 的 adpkg SHA256 `9de5475a…8fe0`，与本地 Downloads 里的文件一致

### 4.2 项目文件

- `docs/` 三份文档已推送
- `src/` 源码已克隆并切到基线提交
- 签名：`src/keystore/lab-release.jks`（4096 位 RSA，10 年，DN `CN=RP4Pro Lab`）
  - 密码保存在 `temp/signing.env`，不要提交、不要外传
- `src/app/build.gradle` 已加 `lab` signingConfig，debug/release 优先用它
- C1/C3 素材已下载到 `temp/upstream/`：
  - MaliBridge v0.5.4.1 / v0.5.4.2 源码 tar.gz
  - Gamescope 3.16.29-p23 组件（SHA256 与官方一致）

### 4.3 构建修复（共 5 处，已写入源码）

1. **CRLF**：Windows 克隆把源码转成 CRLF，Linux 下脚本报 `bash\r`。用 `temp/fix-eol.sh` 批量转 LF（跳过 .bat/.cmd）。
2. **7zip 404**：`src/tools/msitools/packages.txt` 里 `7zip-26.04-1` 已被上游删除，改为 `26.05-1`，SHA256 `17569bafbc5bd1a044ca1db164b96cb15857a22a3575f36679de46e693ae285e`。
3. **WSL 缺工具**：安装 `make`、`patch`、`unzip`。
4. **comet 的 git 安全目录**：`src/tools/comet/build-local.sh` 在容器内 apt 安装 git 之后加 `git config --global --add safe.directory "*"`（注意必须在 apt 之后，放前面会 127）。
5. **Gradle 配置缓存**：首次构建的 APK 用了旧的 testkey 签名，重新打包时加 `--no-configuration-cache` 并确保环境变量齐全。

### 4.4 构建结果

- 完整构建成功：`BUILD SUCCESSFUL in 19m 3s`
- 产物：`src/app/build/outputs/apk/debug/app-debug.apk`，约 72 MB
- 构建日志：`temp/build.log`
- 已验证：
  - 签名证书 `CN=RP4Pro Lab`，SHA-256 `183ee0cef6d56e2cf9ccf2263b6a9841bf641abba4b72352875a00e5855d72dd`，RSA 4096，v2/v3 通过
  - 18:36 的首版 APK 包名正确（`com.rp4pro.droiddeck.lab`）但签名是 testkey
  - 18:44 重打包版签名正确但包名退回 `com.droiddeck.launcher`（没传包名变量）

---

## 5. 当前状态（基线 APK 已在实机运行）

- 最终 APK：`src/app/build/outputs/apk/debug/app-debug.apk`
- 大小：72,353,758 字节（约 69 MiB）
- SHA256：`2f7c66f2febc8af0d4735eb908809660164d7f8630d7a66107d55be9b52be0df`
- 包名：`com.rp4pro.droiddeck.lab`，versionCode 13，versionName 0.4.0
- 签名：`CN=RP4Pro Lab`，SHA-256 `183ee0cef6d56e2cf9ccf2263b6a9841bf641abba4b72352875a00e5855d72dd`，RSA 4096，v2/v3 通过
- 构建结果：`BUILD SUCCESSFUL in 7m 54s`（增量重打包，2026-10-29 18:53）
- 安装命令（待用户确认后执行）：`adb install -r <APK 路径>`

### 实机记录（2026-10-29）

- 19:03 安装成功：`/data/app/...`，dataDir `/data/user/0/com.rp4pro.droiddeck.lab`
- 首次启动弹出权限审核（文件/通知/位置/麦克风/照片视频），用户已授权并继续
- 19:08 Linux 运行时 r9 安装完成，应用数据约 3.1 GB
- 配置页提示"GPU 不受支持"，用户打开"Mali 桥接"后该项消失，进度变为 3/4
- 19:17 Steam Big Picture 启动成功，已登录，最近游戏列表正常显示
- 截图：`temp/screen3.png`（配置页）、`temp/screen5.png`（Steam 界面）
- 日志：vkbridge 正常分配/导出，无 FATAL、无 DEVICE_LOST
- 用户为保持 Steam 地区一致，临时关闭了 RP4 的 VPN
- 19:35 Prison Architect 进入实际画面（CEO 剧情），贴图有闪烁
- 19:44 ForestTD：DroidDeck 内置 HUD 显示 54 fps；摇杆可控制移动；移动时画面闪烁
- ForestTD 参照：Android 原生版 60 fps；DroidDeck 下不开帧生成约 50 fps；开 Win-FG 2x 显示 40→60 fps，但画面跳变
- Steam 这个版本没有内置性能叠加；DroidDeck 自带 PerfHud（fps/电量/温度）需在会话抽屉里手动打开
- 帧生成在 Mali 上不稳定，基线阶段保持关闭
- 19:50 基线验收：ForestTD 连续启动两次正常；Xbox 手柄布局，摇杆/按键可用；RP4 没有 Xbox Guide 键，Steam 键以后用组合键映射；音频待确认
- 包名策略调整：主线 B/C1/C2/C3 统一用 `com.rp4pro.droiddeck.lab` 覆盖升级，保留数据与 Steam 登录；只有 PanVK 用独立包名
- 19:58 Crimsonland（血腥大地）：60 fps，流畅；文字出现小幅偏移/重影（背景菜单文字与数字标签），待 C1/C3 对比
- 19:59 Crimsonland 字体：中文可显示，但不是游戏原生风格字体（疑似 Wine 字体替换/回退），带轻微模糊和偏移；记为已知问题，后续调字体配置或等官方补丁

### C1 合并记录（MaliBridge v0.5.4.2）

- 已把官方 `v0.5.4.1 → v0.5.4.2` 补丁应用到 `src/external/MaliBridge`
- 补丁共 3316 行，大部分干净应用；3 处冲突手工合并：
  - `server/server_memory.c`：保留基线用 `vkb_srv_env()` 的实现（比官方的 `vkb_srv_env_global` 更完整），不采用该 hunk
  - `server/server_instance.c`：加入 `vkb_syncfd_device_init()`，保留基线的 `vkb_cu_device_init()`
  - `server/vkb_server.h`：加入 `vkb_syncfd_device_init/destroy` 声明，保留 cached-upload 声明
- 新增文件：`server/server_apicap.c`、`server/server_syncfd.c` 及对应测试
- 待验证：重新完整构建 → 覆盖安装 → 回归 ForestTD / Prison Architect，确认无回退
- 2026-10-29 20:07 C1 构建成功并覆盖安装（同包名 `com.rp4pro.droiddeck.lab`、lab 签名，数据保留）
- 2026-10-29 20:20 C1 验证：把"配置 → 概览 → 详情 → Steam 菜单"从自动（软件）切成 GPU，重启会话后 Steam 参数变为 `-cef-force-gpu -cef-use-gl=angle -cef-use-angle=vulkan`，库界面 HUD 显示 60 fps（软件模式约 10 fps）
- 但 GPU 模式下 Steam 界面也出现和游戏内一样的画面闪烁；软件模式稳定但只有约 10 fps。闪烁是 Mali 渲染固有问题，C1 未解决，留给 C2/C3
- 待办：C1 下回归 Crimsonland（帧率、文字偏移）与 ForestTD，确认无回退
- 闪烁来源已确认：交接词第 175 行引用 MaliBridge v0.5.4.1 的 RP4 Pro 实测"仍有画面闪烁和伪影"；v0.5.4.2 未承诺修复。old/PanVK 报告中 Crimsonland 闪烁可在 Android 侧 PanVK v1.0.2 消除但降帧（非 DroidDeck 路线）。MaliBridge README 的 green-square artifacts 属 Dimensity 9500，不适用 RP4
- 2026-10-29 20:33 C1 回归：ForestTD 闪烁/跳动明显减少（改善）；Prison Architect 启动失败，报错是 Paradox Launcher 缺失（`launcherpath: File not found`），不是 MaliBridge/C1 代码问题。游戏目录里已有 `Launcher/launcher-installer-windows.msi`，但未安装到 Proton 前缀。修复：桌面模式运行该 msi 安装启动器；或绕过启动器直接运行 `Prison Architect64.exe --launched-from-launcher`
- 2026-10-29 20:36 用户先重装过游戏仍失败（重装不重建 Proton 前缀）；已停止应用并删除 `compatdata/233450`，待重启 Steam 后验证前缀重建与启动器自动安装
- 2026-10-29 20:41 前缀重建后 Prison Architect 恢复正常，进入游戏画面（Day 17 / Sunset 更新），帧数 50–60 fps，闪烁仍在。C1 验收通过：无代码回归，ForestTD 闪烁/跳动有改善，帧率无明显提升（官方版本无性能优化）

### C3 Gamescope 组件（2026-10-29）

- 已把设备 `files/linuxfs` 内的 `gamescope`（usr/local/bin）和 `libVkLayer_FROG_gamescope_wsi_aarch64.so`（usr/lib）替换为 The412Banner 的 `gamescope-3.16.29-p23-nonadreno`
- 替换前备份：`temp/backup-gamescope/`，原哈希 `6e1eb258…` / `47f52dab…`
- 新哈希：`e7524621…` / `bc29c8ca…`，与组件包一致，权限 755
- 待测试：重启 Steam 会话后看闪烁/帧率；回退时用备份文件覆盖回去
- 2026-10-29 20:50 测试结果：Big Picture 与 Prison Architect 闪烁均无改善，性能无提升；Prison Architect 首次启动失败、第二次成功（更像 Proton 前缀首次初始化）
- p23 改动说明：0120–0135 兼容性补丁（驱动拒绝 modifier 时回退 linear、SHM 上传、explicit sync 等），不修闪烁
- 判定无收益，已回退原版 gamescope 与 WSI layer（哈希已校验回 6e1eb258 / 47f52dab）

### C2 官方补丁（进行中）

- 2026-10-29 20:52 已移植 CPU 拓扑（基于官方 4ef6b53e + 9d85af75，手工适配基线脚本）：Windows 看到最快核心优先，写入 WINE_CPU_TOPOLOGY / PROTON_CPU_TOPOLOGY；`DROIDDECK_FASTEST_CORES=0` 可关
- 设备脚本 `files/linuxfs/usr/local/bin/droiddeck-game-env` 已替换，备份 `temp/backup-game-env.py`（原哈希 d93ae2…），新哈希 a4bdbe…；源码同步修改
- 待测试：重启 Steam 会话，日志查 `Windows sees N cores (...)`，对比 CPU 密集型游戏帧率；失败则回退脚本
- 2026-10-29 20:58 发现直接替换设备脚本无效：DroidDeck 每次启动会话会从 APK 重新解压 linuxfs，替换被覆盖（哈希回到 d93ae2…）。改为重新构建 APK 把补丁打进去（构建中）
- 2026-10-29 21:09 重建完成并覆盖安装：包名/签名正确，APK 内 game-env 哈希 d55f1e10…（含 CPU 拓扑 + 调试输出）；待启动会话查 `cpu_topology:` 与 `Windows sees N cores` 日志
- 2026-10-29 21:12 测试发现：补丁只对 Steam 的 explorer.exe 生效（`compatdata/4969280`），实际游戏跑在共享前缀 `compatdata/0`，被 `game_id() is None` 跳过。已修正 fastest 条件：`game_id(env) is not None or SteamGameId.isdigit()`，正在重编 APK
- 2026-10-29 21:20 修正版构建并覆盖安装（签名已校验），待启动 ForestTD 验证游戏进程是否出现 `Windows sees ...`
- 2026-10-29 21:25 游戏进程仍未应用（`compatdata/0` 的 `SteamGameId` 判断不满足）；改为只要 `STEAM_COMPAT_DATA_PATH` 非空就应用最快核心排序（所有 Proton 进程），正在重编
- 后续候选：proot 性能（9294a601 系列）、compositor 性能（8549ac9c/8c4f97a1/067dabe8/b5cc6dd1）、FusionHUD（110cfc28）

---

## 6. 下一步

1. C1：对比官方 v0.5.4.2 与基线内嵌 v0.5.4.1 + rp4pro-fixes 的差异，决定合并范围。
2. 合并 C1 改动、构建、覆盖安装、回归测试。
3. C2：DroidDeck 官方补丁分批移植。
4. C3：Gamescope 组件 A/B。
5. E：PanVK 隔离实验。

---

## 7. 关键命令（复制即用）

所有 WSL 多行命令都要先写成 `.sh` 文件再执行（见第 8 节第 1 条）。

```bash
# 完整基线构建（首次约 19 分钟，之后增量）
MSYS_NO_PATHCONV=1 wsl.exe -d Ubuntu-24.04 -u rp4test -- bash /mnt/c/Users/28157/Desktop/rp4/temp/build-baseline.sh > temp/build.log 2>&1

# 只重新打包 debug（带包名和签名，约 3-5 分钟）
MSYS_NO_PATHCONV=1 wsl.exe -d Ubuntu-24.04 -u rp4test -- bash /mnt/c/Users/28157/Desktop/rp4/temp/rebuild-debug.sh

# 验证 APK（包名/版本/SHA256）
MSYS_NO_PATHCONV=1 wsl.exe -d Ubuntu-24.04 -u rp4test -- bash /mnt/c/Users/28157/Desktop/rp4/temp/verify-apk.sh

# 检查签名证书
MSYS_NO_PATHCONV=1 wsl.exe -d Ubuntu-24.04 -u rp4test -- bash /mnt/c/Users/28157/Desktop/rp4/temp/check-signing.sh

# 签名报告（带密码）
MSYS_NO_PATHCONV=1 wsl.exe -d Ubuntu-24.04 -u rp4test -- bash /mnt/c/Users/28157/Desktop/rp4/temp/signing-report.sh
```

构建脚本 `temp/build-baseline.sh` 会设置：
`ANDROID_HOME=$HOME/android-sdk`、`JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64`、`GITHUB_TOKEN`（读 `temp/gitacc.txt`）、`DROIDDECK_SKIP_RUST=1`、`DROIDDECK_APPLICATION_ID=com.rp4pro.droiddeck.lab`、`DROIDDECK_APP_LABEL=RP4Pro Lab`，并 source `temp/signing.env`。

---

## 8. 交接注意（重要坑）

1. **WSL 多行命令会被 wsl.exe 吞换行**：变量赋值会失效。所有多行脚本必须写成文件，再 `wsl bash /mnt/c/.../xxx.sh`。
2. **Git Bash 路径转换**：调用 WSL 传 `/mnt/c/...` 前加 `MSYS_NO_PATHCONV=1`，否则会被改写成 `C:/Program Files/Git/mnt/...`。
3. **Windows 克隆的源码是 CRLF**：新克隆后先跑 `temp/fix-eol.sh`，否则所有 shell/Python 脚本报 `\r`。
4. **构建环境变量必须齐**：`ANDROID_HOME`、`JAVA_HOME`、`LAB_STORE_PASSWORD`、`LAB_KEY_PASSWORD`、`DROIDDECK_APPLICATION_ID`、`DROIDDECK_APP_LABEL`、`DROIDDECK_SKIP_RUST=1`、`GITHUB_TOKEN`。缺包名变量会退回 `com.droiddeck.launcher`，缺签名密码会用 testkey。
5. **Gradle 配置缓存会带旧签名/包名**：打包时用 `--no-configuration-cache`，或清 `.gradle/configuration-cache`。
6. **签名密码、GitHub 令牌**：只在 `temp/`，不要提交、不要贴聊天。
7. **设备安全**：不要卸载 Eden，不要覆盖其他签名的 APK，不要改系统设置、不要 Root/刷机。
8. **Steam 凭据**：必须用户自己输入，不读取、不导出。
9. **版本号**：当前 APK 显示 `versionCode 13 / versionName 0.4.0`（源码默认），不是 fix43 的 0.6.0；等基线跑通后再决定是否调整版本号规则。
10. **构建输出可能丢失**：Bash 后台任务偶尔不落盘，统一用 `> temp/xxx.log 2>&1` 重定向。

---

## 9. 未完成 / 未验证

- ~~包名 + 签名同时正确的 APK~~（已完成，见第 5 节）
- ~~实机安装、启动、Steam 登录~~（已完成，见第 5 节实机记录）
- ~~游戏实际运行、手柄、退出重进~~（已完成：Prison Architect、ForestTD）
- 音频是否正常（待确认）
- 休眠/恢复（可选）
- 版本号是否要跟 fix43 对齐
- C1：MaliBridge v0.5.4.2 三方合并
- C2：DroidDeck 官方补丁分批移植
- C3：Gamescope 组件 A/B
- E：PanVK v1.1.2 隔离实验
- 源码仓库是否推送到 GitHub（当前 `src/` 只在本地）
- 是否剔除 APK 里用不到的 Turnip 驱动包（Adreno 用，Mali 用不上）

---

## 10. 额度/交接提示

- 环境已完全就绪，重开构建不需要重新装任何东西。
- 上下文丢失时，按顺序读：`step.md` → `docs/项目现状.md` → `docs/整合方案.md`。
- 不要重复已经解决的 5 个构建问题（见 4.3）。
- 不要重新核对已确认的事实（见 4.1），除非怀疑上游更新。
