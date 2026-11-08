# MEIZU 20 Pro — GhostLock profile

- 设备：MEIZU 20 Pro（`meizu_20Pro_CN`），SM8550
- 系统：Flyme 12.6.0.0A / 12.7.0.0A，Android 16
- 内核：`5.15.119-android13-8-gdae8b7f03305-ab1764145455`
- 安全补丁：2026-10-01（12.7）

## 文件

| 文件 | 大小 | SHA-256 | CPU |
| --- | --- | --- | --- |
| `profile-0-1.conf` | `2,951` 字节 | `1e004a6755e53cb823f4efc01b27792fb36b7748ae271cf1c46c54024fefaec6` | `0,1`（推荐） |
| `profile-3-4.conf` | `2,948` 字节 | `1d16249e338298202433c512fc07de0453d6b025f1c2ad67177fd8efb7952a81` | `3,4`（原始成功那次） |

## 重建说明

这两个文件是**重建配置**，不是 TIIEHenry 仓库原始文件的副本。

- 原始来源：[TIIEHenry/Meizu20ProRoot](https://github.com/TIIEHenry/Meizu20ProRoot) 的 `configs/profile.conf`；
- 该仓库没有 LICENSE，默认保留所有权利；本仓库不重新分发其原始文件；
- 原始 `profile.conf` 的 SHA-256：`a656c6366d67b1dbdac6bef62a32d6290ad1fc6114781dbcb2c9f35d33ea978a`；
- 本目录配置的数值来自实机使用记录，文本/注释为本仓库重写，按 CC BY-SA 4.0 授权。

## 导入

1. 把 `.conf` 放到手机存储；
2. 打开 GhostLock → profile → 导入；
3. 选 `profile-0-1.conf`；
4. 确认 UI 里 CPU 核心组是 `0,1`；
5. 执行前清理旧缓存（见 [docs/ghostlock.md](../../docs/ghostlock.md)）。

如果 app 只接受 `.bin`，先用 app 自身的导入/编译功能把 `.conf` 转成内部格式；不要手工改 `.bin`。

## 注意

- 只适用于上面的内核构建；换内核/大版本不要直接用。
- `selected_cpus` 与 `recommended_cpus` 都在文件里；以 app UI 实际选择为准。
- 这是概率路线，失败可能 kernel panic/重启；优先用 [DirtyFrag](../../docs/dirtyfrag.md)。

返回 [profiles/README.md](../README.md) 或 [主 README](../../README.md)。