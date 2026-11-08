# GhostLock（CVE-2026-43499）详细教程

> 适用：MEIZU 20 Pro / Flyme 12.6–12.7 / Android 16 / 内核 `5.15.119-android13-8-gdae8b7f03305-ab1764145455` / 锁定 BL。
> **这是一条概率路线，可能失败并触发 kernel panic/重启。** 优先使用 [DirtyFrag](dirtyfrag.md)。
> 作者已不再在日常使用 GhostLock；本文保留完整的实机记录，供研究和备选。

---

## 1. 原理

GhostLock 利用 **CVE-2026-43499**：futex / rt_mutex 清理路径上的 use-after-free。它不是确定性写，而是竞态——需要把执行时序、CPU 核心、内核偏移都凑对。

成功后通过 KernelSU 的 `ksud` 提供 root 接口。root 只存在于内存，重启消失。

与 DirtyFrag 的关键差异：

- GhostLock 需要**匹配内核的 profile**（`selinux_state`、`init_task`、`init_cred`、`task_struct` 偏移、路由几何等）；
- 需要 Shizuku/LADB/USB 之一，或使用 app 的 **direct 模式**；
- 成功率受内核版本、CPU 核心组、执行方式影响；失败可能 `BUG/BRK` 并重启。

---

## 2. 来源与许可证

- GhostLock 上游：<https://github.com/yukonga/ghostlock-app>（Apache-2.0）
- 魅族 20 Pro 的整套 APK/profile/文档：<https://github.com/TIIEHenry/Meizu20ProRoot>
  - **该仓库未声明许可证**，默认保留所有权利。本指南只链接，不重新分发其 APK、profile 或文档。
  - 作者自称同机实测；没有上游项目为这个仓库背书。
- 5.15 dfroot 模块参考：<https://github.com/qq2070006042-create/yuehong-privilege-assistant>（MPL-2.0）

本仓库镜像/重建的文件：

- APK：[GhostLock-release.apk](../files/ghostlock/GhostLock-release.apk)（上游 pre-release，Apache-2.0）
- APK：[GhostLock_1.2_583.apk](../files/ghostlock/GhostLock_1.2_583.apk)（指南实测构建）
- Profile：[profile-0-1.conf](../profiles/meizu20pro-5.15.119/profile-0-1.conf) / [profile-3-4.conf](../profiles/meizu20pro-5.15.119/profile-3-4.conf)（本仓库重建，CC BY-SA 4.0）
- 来源与许可证说明：[files/ghostlock/README.md](../files/ghostlock/README.md)、[profiles/README.md](../profiles/README.md)

> 如果你要转载/再分发 TIIEHenry 的文件，请先自行确认授权。

---

## 3. 实测环境与成功率

同一台机器：

| 阶段 | 做法 | 结果 |
| --- | --- | --- |
| 12.6 旧内核 | 纯手机 A 方案（Shizuku） | 2/3 成功（695s、1568s；1035s 失败） |
| 12.6 旧内核 | USB 启动 Shizuku | 3 次成功 |
| 12.7 新内核 | `profile-new` + CPU 3,4 | 6 次 1 次成功（约 17%） |
| 12.7 新内核 | CPU 0,1 | 第一次成功 |
| 12.7 新内核 | direct 模式 + `mz20pro_ksu.conf` | 2026-11-07 16:15 一把过，约 25 秒 |

结论：**能用，但不保证一次成功；新内核上明显比旧内核差。** 如果 DirtyFrag 可用，不要用 GhostLock 日常重获 root。

---

## 4. 准备

### 4.1 文件

- GhostLock APK（建议 1.2 系列）；
- 对应内核的 profile（魅族 20 Pro 用 TIIEHenry 仓库版本；不要混用其他机型 profile）；
- Shizuku APK（仅 Shizuku 模式需要）；
- 可选：LADB、Termux（纯手机端 adb 备选）；
- 电脑 + platform-tools（USB 模式/排错）。

### 4.2 手机设置

- 开发者选项 → USB 调试；
- 允许安装未知应用；
- 电量 50% 以上；
- 息屏时间调长，或插电常亮（重启后 `stay_on_while_plugged_in` 会回到 0）；
- 备份重要数据。

### 4.3 先处理缓存

旧缓存会让下一次执行读取错误的内存几何，在固定地址写只读页并触发 panic。

旧路径：

```
/data/local/tmp/ghostlock-app/.ghostlock_iomem
```

新版本运行目录：

```
/data/user/0/com.ghostlock.app/files/.ghostlock_iomem
```

推荐做法：

1. 如果已有旧缓存，先删除或改名；
2. 在 app 私有目录把 `.ghostlock_iomem` 做成 **root 所有的 755 目录**，挡住 app 的缓存写入，让它回退到内置几何；
3. 已成功的缓存可以备份为 `.ghostlock_iomem.good-<时间戳>`。

`reroot.bat` 这类脚本可以在重启后自动清理旧路径并打开 GhostLock；是否使用取决于你的习惯。

---

## 5. Profile

### 5.1 这台机器上解码出的关键值

内核：`5.15.119-android13-8-gdae8b7f03305-ab1764145455`

| 字段 | 值 |
| --- | --- |
| `selinux_state` | `0x2d120f8` |
| `init_task` | `0x2bb2200` |
| `init_cred` | `0x2b6c6f8` |
| `task_struct.real_cred` | `0x790` |
| `task_struct.cred` | `0x798` |
| `task_struct.pi_blocked_on` | `0x8b0` |
| `task_struct.seccomp` | `0x860` |
| `cred` copy size | `176` |
| multicast waiter 几何 | `96 / 264 / 48 / 56` |

这些是**这台机器、这个内核**的值，不要套到其他版本上。

### 5.2 12.7 的 profile 差异

TIIEHenry 仓库只声明支持 Flyme 12.6.0.0A。12.7 上我们曾生成 `profile-new.conf`，只改了两个偏移：

- `security_hook_heads`：`0x20f3580` → `0x20f3600`
- `selinux_blob_sizes`：`0x20f5a08` → `0x20f5a88`

但实机最终发现：**旧的 `profile.conf`（`mz20pro_ksu.conf`）在 12.7 上仍然可用**。这两个 +0x80 的差异不在 W1/W2/W3 的关键路径上，`selinux_state` 两版相同。

所以：

- 不要因为“12.7 没有官方 profile”就直接放弃旧 profile；
- 但也不要把它当成所有 12.7 构建都通用；升级后先核对 `uname -r` 和关键偏移。

### 5.3 CPU 核心组

- 新内核实测 `3,4` 约 1/7；
- 切到 **`0,1 · 2.02 GHz`** 后第一次成功；
- 每次执行前确认 CPU 核心组没有被重置。

---

## 6. Direct 模式（优先）

2026-11-07 起，GhostLock 卸载重装后可以直接在 app 进程内执行，不再经过 Shizuku。

实机日志特征：

```
startup context uid=10253 attr=u:r:untrusted_app:s0
NoNewPrivs=0
Seccomp=2
```

W3 会绕过 seccomp。direct 模式那次全程一把过：W1、W1b、W2、W3 首轮通过，约 25 秒，无 BUG/panic，KernelSU ready。

步骤：

1. 安装 GhostLock；
2. 导入 `mz20pro_ksu.conf`（或你确认过的 profile）；
3. CPU 核心组选 `0,1`；
4. 执行方式选 direct / 直接执行；
5. 上滑露出「执行」按钮（Flyme 手势导航栏可能截断按钮）；
6. 点一次，等待；
7. 成功后验证。

> app 的 prefs 里有 `force_attack_test=true`；日志里可能出现 `SELinux enforce unreadable; assuming enforcing and running W1`，这是正常的，不要关闭该开关。

---

## 7. Shizuku / LADB 模式（备选）

direct 模式彻底不可用时再用。

### 7.1 USB 启动 Shizuku（最稳）

1. 手机连电脑，开启 USB 调试；
2. 通过 Shizuku 的启动脚本/官方说明启动服务；
3. 确认 GhostLock 显示「已就绪」；
4. 执行。

### 7.2 无线调试启动（纯手机）

已验证的 A 方案：

1. 无线调试启动 Shizuku；
2. 确认 GhostLock 已就绪；
3. 关闭无线调试（Wi-Fi 保持）；
4. 设置 → 应用 → Shizuku → 强行停止；
5. 等 1–2 分钟，确认服务仍存活；
6. 打开 GhostLock 执行。

如果出现 `Shizuku UserService did not connect within 3s`，通常是残留了多个 `shizuku_server`。先 kill 掉重复服务，重新启动一个干净的服务，再执行。

### 7.3 LADB / Termux 兜底

- LADB：按 app 说明配对，在本机 shell 里启动 Shizuku；
- Termux：`pkg install android-tools`，用 `adb pair` / `adb connect` 启动；
- 也可以直接执行 Shizuku 的 `libshizuku.so`（路径随版本变化，按当前安装路径取）。

---

## 8. 执行与验证

1. 息屏时间调长/插电常亮；
2. 确认 profile、CPU 核心组、执行方式；
3. 点执行；
4. 成功判据：
   - KernelSU 管理器显示「工作中 [越狱模式]」；
   - `su -c id` 返回 `context=u:r:ksu:s0`；
   - `su -c "/data/adb/ksud -V"` 返回 `ksud 3.3.0 (uapi: 2)`（路径/版本随安装变化）。
5. 失败时设备可能自动重启。先读 `last_kmsg` / dropbox，再决定是否重试。

---

## 9. 失败处理

常见失败签名：

```
BUG/BRK at rt_mutex_adjust_prio_chain+0x984
或 rt_mutex_adjust_prio_chain+0x14b0
调用链：sched_setattr → rt_mutex_adjust_pi
```

这表示 W1 route 的竞态输了，不是 profile 一定错。处理：

1. 等设备重启完成；
2. 确认没有残留的旧缓存；
3. 确认 CPU 核心组是 `0,1`；
4. 再试；连续失败就停手，不要无限重试。

如果 fault 地址是 `kernel_base + 0x16F5990`、PC 在 `rt_mutex_adjust_prio_chain+0x984`，优先检查缓存路径：

- 旧路径：`/data/local/tmp/ghostlock-app/.ghostlock_iomem`（可删/改名）；
- 新路径：`/data/user/0/com.ghostlock.app/files/.ghostlock_iomem`（建议提前做成 root 755 目录挡住）。

---

## 10. 清理

1. 卸载 `com.ghostlock.app`；
2. 删除：
   ```
   /sdcard/Download/ghostlock-debug-log
   /sdcard/Download/ghostlock_profile.conf
   /sdcard/Android/data/com.ghostlock.app
   /data/local/tmp/ghostlock-app
   ```
3. `.ghostlock_iomem` 可能被 `chattr +i` 加了 immutable，需要 root shell：
   ```bash
   chattr -i /data/local/tmp/ghostlock-app/.ghostlock_iomem
   rm -rf /data/local/tmp/ghostlock-app
   ```
4. KernelSU、Shizuku 是独立组件，不要为了清 GhostLock 而卸载它们。

---

## 11. 检测

和 DirtyFrag 一样，KernelSU 的痕迹（`me.weishu.kernelsu`、`ksud`、`u:r:ksu:s0`、`/data/adb/ksu`）比 GhostLock 的包名更显眼。需要过检测时：

- 重启后不要重新提权；
- 不要长期保持 root；
- 银行/支付类 app 用另一台手机。

---

## 12. 引用

- yukonga/ghostlock-app（Apache-2.0）：<https://github.com/yukonga/ghostlock-app>
- TIIEHenry/Meizu20ProRoot（未声明许可证）：<https://github.com/TIIEHenry/Meizu20ProRoot>
- qq2070006042-create/yuehong-privilege-assistant（MPL-2.0）：<https://github.com/qq2070006042-create/yuehong-privilege-assistant>
- CVE-2026-43499：<https://nvd.nist.gov/vuln/detail/CVE-2026-43499>
- KernelSU（GPL-3.0）：<https://github.com/tiann/KernelSU>
- Shizuku（Apache-2.0）：<https://github.com/RikkaApps/Shizuku>

返回 [README](../README.md)。