# 魅族 20 Pro 临时 Root 指南（锁定 BL）

> 在**未解锁 Bootloader** 的 MEIZU 20 Pro 上，通过内核漏洞获得**重启即失效**的临时 root。
> 两条路线：**DirtyFrag（CVE-2026-43284，推荐）** 与 **GhostLock（CVE-2026-43499，备选）**。

本文只记录一台实机上的可复现流程，不保证其他机型、其他 Flyme 版本、其他内核构建可用。执行前请先看完[免责声明](#免责声明)。

---

## 实测环境

| 项目 | 值 |
| --- | --- |
| 设备 | MEIZU 20 Pro（`meizu_20Pro_CN`），SM8550 |
| 系统 | Flyme 12.6.0.0A → 12.7.0.0A，Android 16 |
| 内核 | `5.15.119-android13-8-gdae8b7f03305-ab1764145455` |
| 安全补丁 | 2026-10-01（12.7） |
| Bootloader | 锁定，无 fastboot 恢复 |
| Root 后端 | KernelSU 3.3.0（versionCode 32601），late-load LKM |
| 结果 | DirtyFrag 手动重跑 4/4 成功；GhostLock 概率成功，可能 kernel panic |

> 12.6 与 12.7 的内核构建时间不同，但 `5.15.119-...` 版本号相同；GhostLock 的旧 profile 在 12.7 上仍可用，原因见 [GhostLock 文档](docs/ghostlock.md)。

---

## 结论

- **首选 DirtyFrag。** 页面缓存写原语是确定性的，不需要 profile、不需要 Shizuku、不需要电脑；装 APK，选 KernelSU，点两次，约 10 秒。
- **GhostLock 只作为备选。** 它是 futex/rt_mutex 竞态，需要匹配内核的 profile，可能连续失败，失败时常见 kernel panic/重启。
- 两者都是**临时 root**：只改内存，不写 boot 分区，重启即失效。KernelSU 管理器会一直装着，但 root 状态不会持久化。
- **不要在锁定 BL 的机器上点 KernelSU 的「安装 / 修补 boot」**，会变砖。

---

## 下载与镜像

本仓库镜像了指南用到的上游 APK、可选模块，以及重建的 GhostLock profile：

- [DirtyFrag v1.17 APK](files/dirtyfrag/DirtyFrag-v1.17.apk)（GPL-2.0）
- [GhostLock 上游 pre-release APK](files/ghostlock/GhostLock-release.apk) / [1.2 (583) APK](files/ghostlock/GhostLock_1.2_583.apk)（Apache-2.0）
- [可选 dfroot 5.15 内核模块](files/dirtyfrag/module/dfroot-android13-5.15.ko)（MPL-2.0）
- [GhostLock profile（重建）](profiles/meizu20pro-5.15.119/profile-0-1.conf)
- [全部 SHA-256](SHA256SUMS)
- [二进制许可证与来源说明](files/README.md)

> 二进制文件保留上游许可证，**不**按 CC BY-SA 4.0 重新授权。TIIEHenry 仓库的原始 profile 没有许可证，因此本仓库只提供自己重建的配置，不复制原文件。

---

## 两条路线对比

| | DirtyFrag | GhostLock |
| --- | --- | --- |
| CVE | CVE-2026-43284 | CVE-2026-43499 |
| 原语 | IPsec ESP 页面缓存写（确定性） | futex / rt_mutex UAF（竞态） |
| 需要 profile | 不需要 | 需要，且必须匹配内核偏移 |
| 外部依赖 | 无 | Shizuku / LADB / USB，或 direct 模式 |
| 实测成功率 | 手动重跑 4/4 | 新内核约 1/7；CPU 0,1 + direct 可提高，但不保证 |
| 耗时 | 约 10 秒 | 25 秒到数十分钟，失败要重试 |
| 失败表现 | 实测未出现 panic | 常见 W1 route kernel BUG/panic、自动重启 |
| 重启后 | 失效，手动重跑 | 失效，手动重跑 |
| 开机自动恢复 | Autorun 在 Flyme 上不生效 | 无 |
| 适合 | 日常使用、首次尝试 | 研究、DirtyFrag 不可用时的备选 |

---

## 共同前置

1. **KernelSU 管理器**已安装（包名 `me.weishu.kernelsu`）。不要点「安装 / 修补 boot」。
2. 手机已开启「开发者选项 → USB 调试」；需要 adb 时，电脑装 platform-tools。
3. 允许「安装未知应用」；Flyme 可能还需要按包名放行 USB 安装。
4. 电量 50% 以上，息屏时间调长或插电常亮。
5. 备份重要数据。临时 root 方案不改系统分区，但 kernel panic/重启仍可能造成未保存数据丢失。
6. **不要在拿到 root 后升级系统。** OTA 可能修补漏洞；升级前先确认新内核是否仍可利用。

---

## 快速开始：DirtyFrag（推荐）

完整步骤、原理和排错见 [docs/dirtyfrag.md](docs/dirtyfrag.md)。

1. 下载 `mitschud/DirtyFrag` v1.17 APK，校验：
   - 大小：292,317 字节
   - SHA-256：`e02bd8e11433aa9676d9fb5d4bc7f42825d588015554d5a9e6fdae831a572337`
2. 如果 Flyme 拦截安装（`INSTALL_FAILED_USER_RESTRICTED`），先放行该包名：
   ```bash
   adb shell settings put secure usb_install_item_df.root 'DirtyFrag:1'
   ```
   然后用文件管理器或 `adb install` 安装。
3. 打开 DirtyFrag，允许「读取应用列表」，SU Manager 选 **KernelSU (`me.weishu.kernelsu`)**。
4. 点 **Run exploit**，按钮变成 **Are you sure** 后再点一次。
5. 等待约 10 秒。UI 显示 `100% / Verified / Rooted`，KernelSU 管理器自动弹出并显示「工作中 [越狱模式]」。
6. 验证：见[验证 root](#验证-root)。

重启后 root 消失，打开 DirtyFrag 重复第 4–5 步即可。Autorun (Expert) 在 Flyme 上实测无效，不要依赖。

---

## 快速开始：GhostLock（备选）

完整步骤、profile、失败处理见 [docs/ghostlock.md](docs/ghostlock.md)。这条路线作者已不再使用，仅作为历史记录与备选。

1. 准备 GhostLock APK、对应内核的 profile、Shizuku（或 LADB/USB）。
2. 导入 profile，CPU 核心组选 `0,1`，优先使用 **direct 模式**（不经 Shizuku）。
3. 执行前把息屏时间调长/插电常亮，确认旧缓存已清理。
4. 点执行，等待。成功约 25 秒到数分钟；失败可能 kernel panic 并重启。
5. 成功后用 `su -c id` 或 KernelSU 管理器验证。

---

## 验证 root

- KernelSU 管理器主页显示「工作中 [越狱模式]」。
- 用 KernelSU 的 `libksud.so` 查状态：
  ```bash
  P=$(adb shell pm path me.weishu.kernelsu | sed 's/^package://; s/base.apk$/lib\/arm64\/libksud.so/')
  adb shell "$P" debug info
  ```
  期望包含：
  ```
  version: 32601
  lkm: true
  late_load: true
  runtime_mode: late-load
  ```
- 在 KernelSU 已授权的 app 里执行 `su -c id`，应看到 `context=u:r:ksu:s0`。
- **`adb shell su` 失败是正常的**：root 只授权给 KernelSU 超级用户列表里的 app，shell 的 UID 默认不在列表里。

---

## 重启之后

- 临时 root 立即失效；KernelSU 会显示未工作。
- DirtyFrag：打开 app → Run exploit → Are you sure → 约 10 秒。
- GhostLock：重新导入/确认 profile，重复执行；可能要多轮。
- 两者都无法做到「重启自动恢复」。DirtyFrag 的 Autorun (Expert) 在 Flyme 上不工作；GhostLock 没有可靠的 boot 触发路径。
- 如果只是想取消 root，直接重启即可，不需要卸载 KernelSU。

---

## 检测与隐藏

临时 root 仍然是 root。银行、支付、部分游戏和反作弊会检测：

- 已安装包：`df.root`、`me.weishu.kernelsu`，以及任何包含 `root` / `kernelsu` / `magisk` 的包名；
- 运行痕迹：`ksud` 守护进程、`u:r:ksu:s0` 域、`/data/adb/ksu`、late-load 内核模块；
- app 自己尝试执行 `su`、读 `/proc`、检查挂载命名空间等。

实际建议：

- **需要过检测时不要保持 root。** 重启后不重新提权，是最可靠的「隐藏」。
- DirtyFrag 拿到 root 后可以卸载 app（不影响当前 root），APK 留在手机上，重启后再装。重装后 Android UID 会变，KernelSU 里如提示授权就重新给。
- 改包名重打包只能绕过包名扫描，躲不过 KernelSU 自身痕迹和 `ksud`，不值得。
- 银行/支付类 app 建议直接用另一台手机；这套临时 root 不是为对抗强检测设计的。

---

## 清理

### DirtyFrag

- 桌面长按 → 卸载（不需要 root）。当前 root 不会立刻消失，ksud 和内核模块已在内存里；重启后彻底清空。
- 保留 `DirtyFrag-v1.17.apk`，下次重启后重装。
- KernelSU 超级用户列表里可能留下失效条目，不影响使用，可在管理器里手动删。

### GhostLock

- 卸载 `com.ghostlock.app`。
- 删除 `/sdcard/Download/ghostlock-debug-log`、`/sdcard/Download/ghostlock_profile.conf`、`/sdcard/Android/data/com.ghostlock.app`。
- 删除 `/data/local/tmp/ghostlock-app`。其中 `.ghostlock_iomem` 可能被 `chattr +i` 加了 immutable，需要 root shell：
  ```bash
  chattr -i /data/local/tmp/ghostlock-app/.ghostlock_iomem
  rm -rf /data/local/tmp/ghostlock-app
  ```
- KernelSU、Shizuku 是独立组件，不要为了清 GhostLock 而卸载它们。

---

## 文档索引

- [DirtyFrag 详细教程](docs/dirtyfrag.md)
- [GhostLock 详细教程](docs/ghostlock.md)
- [排错与常见问题](docs/troubleshooting.md)
- [开源、引用与许可证](docs/references.md)

---

## 开源、引用与致谢

本仓库是**文档**，不是 exploit 发行版：不包含漏洞利用源码、不镜像 APK、不附带内核模块。所有工具请从上游项目获取。

- 本文档采用 **CC BY-SA 4.0** 许可，见 [LICENSE](LICENSE)。
- 转载、翻译、引用请注明本仓库链接与作者，并保留相同许可。
- 第三方项目、APK、内核模块遵循其各自许可证；本仓库不重新分发它们。

主要上游与引用：

| 项目 | 用途 | 许可证 |
| --- | --- | --- |
| [mitschud/DirtyFrag](https://github.com/mitschud/DirtyFrag) | DirtyFrag APK / `dfroot.ko` | GPL-2.0 |
| [diabl0w/DFRoot](https://github.com/diabl0w/DFRoot) | DirtyFrag 上游 | GPL-2.0 |
| [yukonga/ghostlock-app](https://github.com/yukonga/ghostlock-app) | GhostLock APK | Apache-2.0 |
| [TIIEHenry/Meizu20ProRoot](https://github.com/TIIEHenry/Meizu20ProRoot) | 魅族 20 Pro GhostLock profile/文档 | 未声明许可证（仅链接，不再分发） |
| [qq2070006042-create/yuehong-privilege-assistant](https://github.com/qq2070006042-create/yuehong-privilege-assistant) | 5.15 dfroot 模块参考 | MPL-2.0 |
| [tiann/KernelSU](https://github.com/tiann/KernelSU) | Root 后端 | GPL-3.0 |
| [RikkaApps/Shizuku](https://github.com/RikkaApps/Shizuku) | GhostLock 的 shell 入口 | Apache-2.0 |
| [tytydraco/LADB](https://github.com/tytydraco/LADB) | 纯手机端 adb 备选 | 见仓库 |

漏洞公告：

- [CVE-2026-43284](https://nvd.nist.gov/vuln/detail/CVE-2026-43284) — Dirty Frag（IPsec ESP 页面缓存写）
- [CVE-2026-43500](https://nvd.nist.gov/vuln/detail/CVE-2026-43500) — Dirty Frag 关联变体（RxRPC）
- [CVE-2026-46300](https://nvd.nist.gov/vuln/detail/CVE-2026-46300) — Fragnesia
- [CVE-2026-43499](https://nvd.nist.gov/vuln/detail/CVE-2026-43499) — GhostLock 使用的 futex/rt_mutex 漏洞

感谢上述项目的作者与维护者。没有他们的公开工作，这份文档不会存在。

---

## 免责声明

- 本指南仅用于**你自己的设备**、安全研究与学习。不要用于你无权控制的设备。
- 临时 root 会扩大攻击面；root 期间任何 app 获得授权都可能带来风险。
- 内核漏洞利用可能导致 **kernel panic、重启、数据丢失、系统损坏、保修失效**。作者不对任何后果负责。
- 不同机型、不同内核、不同 Flyme 版本的风险不同；不要照搬。先确认内核版本和 profile 匹配。
- 请遵守当地法律法规与设备厂商条款。