# GhostLock profiles

## 许可与来源

本目录的 `profile-0-1.conf` 和 `profile-3-4.conf` 是**本仓库重建**的配置文件：

- 数值来自 2026-11-08 在 MEIZU 20 Pro（`5.15.119-android13-8-gdae8b7f03305-ab1764145455`）上实际使用的 profile；
- 原始 profile 由 [TIIEHenry/Meizu20ProRoot](https://github.com/TIIEHenry/Meizu20ProRoot) 发布，该仓库**没有 LICENSE 文件**，默认保留所有权利；
- 因此本仓库**不重新分发**该仓库的 `profile.conf` / `profile-source.conf` / `profile.bin`，只提供我们自己重建的文本配置，并标注来源与校验值；
- 重建配置采用 **CC BY-SA 4.0**，与本文档其余部分一致。

如果你要使用原始文件，请直接从 TIIEHenry 仓库获取，并自行确认其授权条件。原始 `configs/profile.conf` 的 SHA-256 是：

```
a656c6366d67b1dbdac6bef62a32d6290ad1fc6114781dbcb2c9f35d33ea978a
```

你可以用它校验自己从上游拿到的文件。

## 可用配置

### `meizu20pro-5.15.119/profile-0-1.conf`

- 推荐变体；
- `selected_cpus` 为 `0,1`；
- 与指南中“CPU 核心组选 0,1”一致。

### `meizu20pro-5.15.119/profile-3-4.conf`

- 原始成功那次使用的 CPU 组合；
- `selected_cpus` 为 `3,4`；
- 想复现 2026-11-07 16:15 那次 direct 运行，用这个。

两个文件的其他数值相同。

## 适用范围

**只适用于：**

- MEIZU 20 Pro（`meizu_20Pro_CN`）
- 内核 `5.15.119-android13-8-gdae8b7f03305-ab1764145455`
- Flyme 12.6.0.0A / 12.7.0.0A，Android 16

**不要**套用到其他内核、其他机型或其他 Flyme 大版本。偏移不对可能直接 kernel panic。

## 导入

1. 把对应的 `.conf` 放进手机存储；
2. 打开 GhostLock，进入 profile 选择/导入；
3. 选择 `profile-0-1.conf`（或 `profile-3-4.conf`）；
4. 确认 UI 里的 CPU 核心组与文件一致；
5. 执行前清理旧缓存，见 [docs/ghostlock.md](../docs/ghostlock.md)。

GhostLock 可能会把 `.conf` 编译成内部的 `.bin`；以 app 实际导入结果为准。

## 关键值速查

| 字段 | 值（十进制） | 十六进制 |
| --- | --- | --- |
| `offset.init_task` | 45818368 | `0x2BB2200` |
| `offset.init_cred` | 45532920 | `0x2B6C6F8` |
| `offset.selinux_enforcing` | 47259896 | `0x2D120F8` |
| `offset.security_hook_heads` | 34551168 | `0x20F3580` |
| `offset.selinux_blob_sizes` | 34560520 | `0x20F5A08` |
| `task_struct.real_cred` | 1936 | `0x790` |
| `task_struct.cred` | 1944 | `0x798` |
| `task_struct.pi_blocked_on` | 2224 | `0x8B0` |
| `task_struct.seccomp` | 2144 | `0x860` |
| `cred.copy_size` | 176 | `0xB0` |

注意：12.7 的 `security_hook_heads` / `selinux_blob_sizes` 在另一份 profile 里曾出现 +0x80 的差异；本配置保留的是实际使用并成功的那组值。升级系统后先核对 `uname -r` 和关键偏移。

返回 [README](../README.md)。