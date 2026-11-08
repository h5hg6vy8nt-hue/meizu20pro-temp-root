# DirtyFrag（CVE-2026-43284）详细教程

> 适用：MEIZU 20 Pro / Flyme 12.x / Android 16 / 内核 `5.15.119-android13-8-gdae8b7f03305-ab1764145455` / 锁定 BL。
> 其他机型、内核、Flyme 版本未验证。先读 [README](../README.md) 的环境与免责声明。

---

## 1. 为什么选它

DirtyFrag 利用的是 **CVE-2026-43284（Dirty Frag）**，一个 IPsec ESP 接收路径上的页面缓存写原语。

和 GhostLock 的 futex/rt_mutex 竞态不同，它是确定性的：同样的环境、同样的步骤，成功与否不靠抽奖。实机手动重跑 **4/4**，每次约 **10 秒**，没有出现 kernel panic。

它也不需要 GhostLock 那种「内核偏移 profile」：

- APK 的 `libdfroot.so` 内置了多组预编译内核模块，覆盖 `5.10.245`、`5.10.252`、`5.15.202`、`6.1.166`、`6.6.127`、`6.12.76`、`6.18.32` 等 vermagic；
- 启动时读 `uname -r` 和 Android 版本，没有精确匹配就选最接近的模块；
- 模块运行时从 `/sys/kernel/btf/vmlinux` 解析符号，不依赖用户提供 offset。

所以你的 `5.15.119-android13-8` 和模块自带的 `5.15.202-android13-5.15.202_r00-dirty` 不一致，它仍然能完成加载。**不要只看 vermagic 就判断不可用。**

---

## 2. 原理（实测观察到的那条链）

1. `libdfroot.so`（exploit 引擎）通过 `IpSecManager` 建立 ESP/UDP 封装 socket，用 `vmsplice` + `splice` 把目标文件的 page 引用送进 UDP 发送路径。
2. CVE-2026-43284 让内核在 ESP 处理时写进目标文件的 page cache。目标是 `/apex/com.android.runtime/bin/crash_dump64`；磁盘文件不动，只改内存页。
3. 触发一次进程 core dump，init 以 root 拉起 `crash_dump64`。实际执行的是被改过的页缓存，payload（`splicehelper`）在特权上下文里跑起来。
4. payload 用 `init_module` 加载内置的 `dfroot.ko`。模块：
   - 从 BTF 自动解析符号；
   - 把 `selinux_state` 改为 permissive，patch `avc_has_perm`、`avc_has_perm_noaudit`；
   - patch 厂商 LSM 钩子（`task_defex_*`、`oplus_*` 等，针对三星/OPPO 系加固，魅族上未命中也不影响）；
   - 用 `call_usermodehelper` 以 root 启动 `bootstrap`。
5. `bootstrap` 从 `/data/user_de/0/df.root/ksud` 启动 SU daemon。**v1.17 不再自带 ksud**，它是从你选中的 SU Manager 的 `libksud.so` 现场 stage 的；manager 不带这个库就给不了 root。
6. KernelSU 以 **late-load LKM** 模式接管：`lkm:true`、`late_load:true`、`runtime_mode:late-load`。
7. 模块最后用 `invalidate_inode_pages2` 清掉 `crash_dump64` 的脏页，磁盘上的二进制恢复原样。root 状态留在内存里，重启消失。

> 本仓库不包含上述 exploit 代码或内核模块，只记录流程与公开来源。

---

## 3. 下载与校验

- 项目：<https://github.com/mitschud/DirtyFrag>
- 版本：v1.17（`versionCode` 34，`versionName` 1.17）
- APK 大小：`292,317` 字节
- SHA-256：`e02bd8e11433aa9676d9fb5d4bc7f42825d588015554d5a9e6fdae831a572337`

校验：

```bash
sha256sum DirtyFrag-v1.17.apk
```

Windows PowerShell：

```powershell
Get-FileHash .\DirtyFrag-v1.17.apk -Algorithm SHA256
```

哈希不一致就不要装。

---

## 4. 安装

### 4.1 正常安装

把 APK 放进手机存储，用文件管理器打开安装。需要允许「安装未知应用」。

### 4.2 Flyme 拦截时

Flyme 可能报：

```
INSTALL_FAILED_USER_RESTRICTED
```

它有一套按包名的 USB 安装白名单。先放行 `df.root`：

```bash
adb shell settings put secure usb_install_item_df.root 'DirtyFrag:1'
```

确认：

```bash
adb shell settings get secure usb_install_item_df.root
# DirtyFrag:1
```

然后再用文件管理器安装，或：

```bash
adb install -r DirtyFrag-v1.17.apk
```

如果仍然被拦，检查：

- 开发者选项里的「USB 安装」是否允许；
- 是否用了 Flyme 的「应用安装器」而不是第三方安装器；
- 安装来源是否被「纯净模式」/安全中心拦截。

> 这条白名单设置会保留；以后卸载重装 DirtyFrag 不需要再执行一次。

---

## 5. 首次运行

1. 打开 **DirtyFrag**（包名 `df.root`）。
2. 授予「读取应用列表」权限。它需要扫描已安装的 SU Manager；拒绝的话会显示 `no SU manager selected`。
3. 在 **SU Manager** 卡片里选择 **KernelSU (`me.weishu.kernelsu`)**。
   - 必须选一个自带 `libksud.so` 的 manager；
   - 已实测 KernelSU 3.3.0（32601）可用；
   - 换 manager 后要重新选。
4. 可选：打开 **Expert Mode**。它会显示更多日志/选项。
5. **不要打开 Autorun。** 在 Flyme 上实测无效，见第 8 节。

---

## 6. 执行

1. 点底部 **Run exploit**。
2. 按钮变成 **Are you sure** 后再点一次。
3. 等待约 10 秒。正常表现：
   - UI 显示 `100%`、`Verified`、`Rooted`；
   - KernelSU 管理器自动弹出；
   - KernelSU 主页显示「工作中 [越狱模式]」。
4. 如果 app 显示 `Running` 但一直不动，等 30 秒；仍不动就强制停止 DirtyFrag，重开再点一次。

---

## 7. 验证

### 7.1 KernelSU 界面

KernelSU 管理器主页显示「工作中 [越狱模式]」；超级用户列表里的 app 可以正常申请 root。

### 7.2 命令行验证

用 KernelSU 自带的 `libksud.so`：

```bash
P=$(adb shell pm path me.weishu.kernelsu | sed 's/^package://; s/base.apk$/lib\/arm64\/libksud.so/')
adb shell "$P" debug info
```

期望输出：

```
version: 32601
flags: 0x5
uapi_version: 2
features: 0x6
lkm: true
late_load: true
runtime_mode: late-load
pr_build: false
```

注意：

- **`adb shell su` 失败是正常的。** KernelSU 只给超级用户列表里的 app 授权，shell UID 默认不在列表里。
- 要验证 `su`，请在已授权的 app 里执行，或使用 KernelSU 管理器里的终端。

---

## 8. 重启之后

- root 立即失效，KernelSU 显示未工作。
- DirtyFrag 的 **Autorun (Expert)** 在 Flyme 上**不生效**：
  - Expert Mode 已开；
  - Autorun 开关已开；
  - Flyme「应用信息 → 耗电和后台 → 手动管理 → 允许自启动」已勾选；
  - `df.root_op_23/24` 也改过 4；
  - 重启后 logcat 仍没有 `df.root.BootReceiver` 启动记录，KernelSU 仍是 `lkm:false`。
- 正确做法：重启后手动打开 DirtyFrag → Run exploit → Are you sure。约 10 秒。
- 不需要电脑、不需要 Shizuku、不需要无线调试。

---

## 9. 卸载、隐藏与检测

### 9.1 拿到 root 后卸载 app

DirtyFrag 只负责提权，root 起来之后 app 本身可以卸载：

- 桌面长按 → 卸载；
- 不需要 root；
- 当前 root 不会立刻消失，ksud 和内核模块已经在内存里；
- 重启后 root 清空，需要时再从 APK 重装。

重装后 Android UID 会变，KernelSU 里如果弹授权就重新给一次。

### 9.2 检测面

`df.root` 这个包名本身就是红旗：

- 有 `QUERY_ALL_PACKAGES` 的 app 可以直接扫到；
- root 检测库会按 `root`、`magisk`、`kernelsu`、`supersu` 等关键词过滤；
- 更致命的是 `me.weishu.kernelsu`、`ksud`、`u:r:ksu:s0`、`/data/adb/ksu`、late-load LKM。这些是 KernelSU 留下的，改 DirtyFrag 包名没用。

Play Integrity 方面：BL 没解、启动链是干净的，设备完整性未必挂；但银行/支付/反作弊 app 自带的 root 检测会看包名和运行痕迹。

### 9.3 实用建议

- 需要过检测时，**重启后不要重新提权**；这是最可靠的隐藏。
- 拿完 root 就卸载 DirtyFrag，APK 留在手机里备用。
- 不要为了隐藏去重打包 APK；收益只覆盖包名扫描，KernelSU 痕迹还在。
- 银行、支付、部分游戏建议用另一台手机。

---

## 10. 故障排查

| 现象 | 处理 |
| --- | --- |
| 安装报 `INSTALL_FAILED_USER_RESTRICTED` | 执行 `settings put secure usb_install_item_df.root 'DirtyFrag:1'` 后重装 |
| `no SU manager selected` | 安装 KernelSU；授予读取应用列表；重开 app |
| 选不到 KernelSU | 确认 KernelSU 已安装且未被停用；重启 app；检查 manager 是否自带 `libksud.so` |
| 一直 `Running` | 等 30 秒；强制停止重开；再点一次 |
| 点了没反应 | 确认 SU Manager 仍是 KernelSU；杀掉 app 重开 |
| 显示 `Unsupported kernel x.y android z - no embedded module` | app 没有该内核的精确模块；会尝试 nearest，但不保证；换版本/等上游 |
| root 成功但 `adb shell su` 不行 | 正常，shell 未授权；用已授权 app 或 KernelSU 终端 |
| 重启后 root 没了 | 正常；手动重跑 |
| 开机 Autorun 不工作 | Flyme 上已知无效；手动重跑 |
| 执行后重启/panic | 先读 last_kmsg/dropbox；不要连续盲试；确认机型/内核是否被上游支持 |

更多通用问题见 [troubleshooting.md](troubleshooting.md)。

---

## 11. 安全与法律

- 只在你自己的设备上使用。
- root 后只给可信 app 授权；KernelSU 的超级用户列表就是攻击面。
- 不要在有重要账号/支付的手机上长期保持 root。
- 内核漏洞利用存在数据丢失、系统损坏、保修失效的风险；作者不承担后果。
- 遵守当地法律与厂商条款。

---

## 12. 引用

- mitschud/DirtyFrag（GPL-2.0）：<https://github.com/mitschud/DirtyFrag>
- diabl0w/DFRoot（GPL-2.0）：<https://github.com/diabl0w/DFRoot>
- CVE-2026-43284：<https://nvd.nist.gov/vuln/detail/CVE-2026-43284>
- CVE-2026-43500：<https://nvd.nist.gov/vuln/detail/CVE-2026-43500>
- CVE-2026-46300（Fragnesia）：<https://nvd.nist.gov/vuln/detail/CVE-2026-46300>
- KernelSU（GPL-3.0）：<https://github.com/tiann/KernelSU>

返回 [README](../README.md)。