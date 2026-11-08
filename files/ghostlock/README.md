# GhostLock APK

本目录有两个构建：

| 文件 | 大小 | SHA-256 | 说明 |
| --- | --- | --- | --- |
| `GhostLock-release.apk` | `2,529,411` 字节 | `ac72500d3003ab810d46839dae19ca90ea277707b6bc433cb7c6006e1114c4c7` | 上游 pre-release（2026-09-17） |
| `GhostLock_1.2_583.apk` | `2,514,464` 字节 | `c77b4243bd29500119a046e1ae471120058a3a17621d29a689dacb401ea29212` | 指南实测的 1.2 (583) 构建 |

- 上游：[YuKongA/ghostlock-app](https://github.com/YuKongA/ghostlock-app)
- 许可证：**Apache-2.0**（见 `LICENSE-APACHE-2.0.txt`）

## 来源说明

- `GhostLock-release.apk` 直接来自上游 release：
  <https://github.com/YuKongA/ghostlock-app/releases/tag/pre-release>
- `GhostLock_1.2_583.apk` 是上游 GhostLock 的 1.2 (583) 构建，镜像自
  [TIIEHenry/Meizu20ProRoot](https://github.com/TIIEHenry/Meizu20ProRoot) 的 `apks/` 目录。
  - TIIEHenry 仓库本身**未声明许可证**。
  - 这里只镜像了该 APK；没有复制该仓库的 profile、脚本、文档或其他文件。
  - APK 本身的代码/资源归 GhostLock 上游，按 Apache-2.0 分发。

两个 APK 均为**未修改**副本；没有重新签名或注入代码。

## Apache-2.0 归属

GhostLock 由 YuKongA 及贡献者开发，按 Apache License 2.0 授权。分发时请保留：

- `LICENSE-APACHE-2.0.txt`
- 本 README 中的来源与归属说明
- 上游 NOTICE（如上游提供）

上游仓库：<https://github.com/YuKongA/ghostlock-app>

## Profile 不在本目录

GhostLock 的 Meizu 20 Pro profile **不是**上游 APK 的一部分，也没有随 APK 一起镜像。原因和可用配置见：

- [profiles/README.md](../../profiles/README.md)
- [profiles/meizu20pro-5.15.119/README.md](../../profiles/meizu20pro-5.15.119/README.md)

简单说：TIIEHenry 仓库的原始 profile 没有许可证，本仓库不重新分发它；我们提供了**自己重建**的 `profile-0-1.conf` / `profile-3-4.conf`，并注明了原始来源与校验值。

## 使用

完整步骤见 [docs/ghostlock.md](../../docs/ghostlock.md)。注意这是概率路线，可能失败并触发 kernel panic/重启；优先使用 DirtyFrag。

返回 [files/README.md](../README.md) 或 [主 README](../../README.md)。