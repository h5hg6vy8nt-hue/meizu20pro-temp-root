# 排错与常见问题

> 先确认你读的是对应路线的文档：[DirtyFrag](dirtyfrag.md) / [GhostLock](ghostlock.md)。

---

## 1. 基础检查

```bash
adb devices
# 应看到 <serial>  device；offline/unauthorized 先处理授权

adb shell getprop ro.product.device
adb shell getprop ro.build.display.id
adb shell getprop ro.build.version.security_patch
adb shell uname -r
```

对照 [README](../README.md) 的实测环境。内核版本不同，GhostLock 的 profile 不能直接用；DirtyFrag 也可能遇到 `no embedded module`。

---

## 2. Root 验证

### 2.1 KernelSU 界面

主页显示「工作中 [越狱模式]」即已工作。

### 2.2 `libksud.so debug info`

```bash
P=$(adb shell pm path me.weishu.kernelsu | sed 's/^package://; s/base.apk$/lib\/arm64\/libksud.so/')
adb shell "$P" debug info
```

`lkm: true` + `late_load: true` + `runtime_mode: late-load` 表示 late-load 模块已加载。

### 2.3 `su -c id`

在 KernelSU 已授权的 app 里执行，期望：

```
uid=0(root) ... context=u:r:ksu:s0
```

### 2.4 为什么 `adb shell su` 不行

KernelSU 只给超级用户列表里的 app 授权。shell 的 UID 不在列表里，所以 `su` 被拒绝。这不是 root 没成功。

不要为了这个把 shell 加进授权列表；需要命令行时用 KernelSU 管理器里的终端，或已授权的文件管理器/终端 app。

---

## 3. DirtyFrag 常见问题

| 现象 | 原因/处理 |
| --- | --- |
| `INSTALL_FAILED_USER_RESTRICTED` | Flyme 按包名拦截。执行 `settings put secure usb_install_item_df.root 'DirtyFrag:1'` 后重装 |
| 安装成功但打不开 | 检查是否被「纯净模式」/安全中心限制；允许安装未知应用 |
| `no SU manager selected` | 授予「读取应用列表」；安装 KernelSU；重开 app |
| SU Manager 列表为空 | KernelSU 未安装/被停用；或 manager 不带 `libksud.so` |
| 一直 `Running` | 等 30 秒；强制停止重开；确认 SU Manager 仍是 KernelSU |
| 显示 `Unsupported kernel ...` | app 没有精确模块，会尝试 nearest；不保证成功。查上游 release 是否新增模块 |
| 成功后重启失效 | 正常。临时 root 不持久化；手动重跑 |
| Autorun 不工作 | Flyme 上已知无效；不要依赖 |
| 执行后 panic | 记录 `last_kmsg`/dropbox；不要连续盲试；确认内核是否被上游支持 |

---

## 4. GhostLock 常见问题

### 4.1 `Shizuku UserService did not connect within 3s`

通常是残留了多个 `shizuku_server`：

```bash
adb shell ps -A | grep shizuku
```

kill 掉重复服务，重新启动一个干净的 Shizuku，再执行。不要急着重启手机。

### 4.2 执行后自动重启 / W1 BUG

常见签名：

```
rt_mutex_adjust_prio_chain+0x984
或 +0x14b0
sched_setattr → rt_mutex_adjust_pi
```

这是竞态失败，不一定是 profile 错。处理顺序：

1. 确认 CPU 核心组是 `0,1`；
2. 清理旧缓存；
3. 再试；
4. 连续失败就停手。

### 4.3 缓存导致固定地址 panic

如果 fault 地址是 `kernel_base + 0x16F5990`，优先检查：

- `/data/local/tmp/ghostlock-app/.ghostlock_iomem`
- `/data/user/0/com.ghostlock.app/files/.ghostlock_iomem`

旧路径可以删/改名；新路径建议提前做成 root 所有 755 目录挡住。

### 4.4 profile 不匹配

- 确认 `uname -r` 与 profile 来源一致；
- 不要用其他机型的 profile；
- 12.7 上旧 `mz20pro_ksu.conf` 可能仍可用，但先核对关键偏移；
- 改 profile 后重新导入，不要只改文件名。

### 4.5 Shizuku 无线调试不稳定

优先 USB；其次纯手机 A 方案（无线启动 → 关无线 → force-stop Shizuku app）；最后 LADB/Termux。无线路径在实机上不如 USB 稳。

### 4.6 执行按钮被手势栏截断

Flyme 手势导航会挡住 app 底部按钮。先上滑露出按钮，再点。

---

## 5. KernelSU 相关

- **不要点「安装 / 修补 boot」**。锁定 BL 的机器上这会变砖。
- KernelSU 管理器可以一直装着；root 状态由内存中的模块决定，重启后 `lkm:false` 是正常的。
- 卸载/重装 DirtyFrag 会换 Android UID，KernelSU 里可能需要重新授权。
- 超级用户列表里卸载 app 后的失效条目不影响使用，可手动删除。
- 不要为了隐藏而刷模块/改分区；这台机器没有 fastboot 恢复。

---

## 6. 日志与 panic 排查

能拿到 root 时：

```bash
adb shell su -c "dmesg | tail -200"
adb shell su -c "ls -la /sys/fs/pstore"
adb shell su -c "ls -la /data/system/dropbox | tail"
```

拿不到 root 时：

```bash
adb shell logcat -d -b all | tail -300
adb shell dumpsys dropbox --print | tail -200
```

重点看：

- `BUG:` / `Unable to handle kernel` / `pc :`；
- `rt_mutex_adjust_prio_chain` / `sched_setattr`；
- `dfroot` / `ksud` / `KernelSU`；
- `avc:` 拒绝（SELinux）。

---

## 7. 检测与隐藏（简版）

- 包名：`df.root`、`me.weishu.kernelsu`；
- 运行痕迹：`ksud`、`u:r:ksu:s0`、`/data/adb/ksu`、late-load LKM；
- Play Integrity：BL 锁定、启动链干净时设备完整性未必挂，但 app 自带检测会看包名/运行痕迹；
- 最可靠的隐藏：**重启后不提权**；
- 拿完 root 卸载 DirtyFrag；APK 留着；
- 银行/支付用另一台手机。

---

## 8. 不要做的事

- 不要在锁定 BL 的机器上刷 boot/修补 boot；
- 不要用其他机型/内核的 profile；
- 不要连续盲试导致反复 panic；
- 不要在 root 状态下登录重要账号；
- 不要升级系统后再套用旧结论；OTA 可能修补漏洞；
- 不要把 TIIEHenry 仓库的文件当成有许可证的素材再分发。

---

## 9. 还是不行

收集以下信息再求助/开 issue：

1. 机型、Flyme 版本、`uname -r`、安全补丁；
2. 走的哪条路线、app 版本、profile 名称；
3. 完整报错/日志（脱敏）；
4. 是否重启过、是否清过缓存；
5. KernelSU 的 `debug info` 输出。

返回 [README](../README.md)。