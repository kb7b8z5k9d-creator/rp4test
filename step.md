# RP4 Pro DroidDeck — 项目进度（step.md）

最后更新：2026-10-29 18:56（东八区）
用途：额度耗尽、上下文压缩或换模型接手时，先读这份文件恢复现场。

---

## 0. 一句话现状

基线 APK 已完成并通过验证：包名 `com.rp4pro.droiddeck.lab`、自建签名 `CN=RP4Pro Lab`、可安装。下一步是装到 RP4 Pro 实机测试。

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

## 5. 当前状态（基线 APK 已完成，待实机安装）

- 最终 APK：`src/app/build/outputs/apk/debug/app-debug.apk`
- 大小：72,353,758 字节（约 69 MiB）
- SHA256：`2f7c66f2febc8af0d4735eb908809660164d7f8630d7a66107d55be9b52be0df`
- 包名：`com.rp4pro.droiddeck.lab`，versionCode 13，versionName 0.4.0
- 签名：`CN=RP4Pro Lab`，SHA-256 `183ee0cef6d56e2cf9ccf2263b6a9841bf641abba4b72352875a00e5855d72dd`，RSA 4096，v2/v3 通过
- 构建结果：`BUILD SUCCESSFUL in 7m 54s`（增量重打包，2026-10-29 18:53）
- 安装命令（待用户确认后执行）：`adb install -r <APK 路径>`

---

## 6. 下一步

1. 验证新 APK 的包名、签名、SHA256（用 `temp/verify-apk.sh`、`temp/check-signing.sh`）。
2. 征得用户同意后，`adb install -r` 安装到这个独立包名（不会覆盖 Eden 或其他应用）。
3. 启动应用，让用户自己登录 Steam，做基线测试（启动、游戏模式、手柄、音频、退出重进）。
4. 记录构建/测试结果到文档，推送。
5. 之后按 C1 → C2 → C3 → E 推进。

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
- 实机安装、启动、Steam 登录与游戏测试
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
