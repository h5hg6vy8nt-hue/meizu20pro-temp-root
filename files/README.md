# 二进制与配置文件

本目录存放指南用到的 APK、可选内核模块和许可证文本。**这些文件不是本仓库原创**，各自保留上游许可证；本仓库的 CC BY-SA 4.0 只适用于文档和 `profiles/` 中由我们重建的配置。

| 路径 | 内容 | 来源 | 许可证 |
| --- | --- | --- | --- |
| `dirtyfrag/DirtyFrag-v1.17.apk` | DirtyFrag v1.17 | [mitschud/DirtyFrag](https://github.com/mitschud/DirtyFrag) | GPL-2.0 |
| `dirtyfrag/module/dfroot-android13-5.15.ko` | 可选 5.15 内核模块 | [yuehong-privilege-assistant](https://github.com/qq2070006042-create/yuehong-privilege-assistant) | MPL-2.0 |
| `ghostlock/GhostLock-release.apk` | 上游 GhostLock pre-release | [YuKongA/ghostlock-app](https://github.com/YuKongA/ghostlock-app) | Apache-2.0 |
| `ghostlock/GhostLock_1.2_583.apk` | 指南实测的 1.2 (583) 构建 | 上游 GhostLock；镜像自 [TIIEHenry/Meizu20ProRoot](https://github.com/TIIEHenry/Meizu20ProRoot) | Apache-2.0（APK 本身） |

- 所有文件均为**未修改**的上游构建；没有重新签名、没有注入代码。
- DirtyFrag 的完整对应源码：<https://github.com/mitschud/DirtyFrag>（tag `v1.17`）。GPL-2.0 要求的分发声明见 `dirtyfrag/README.md`。
- GhostLock 的完整源码：<https://github.com/YuKongA/ghostlock-app>。Apache-2.0 的 NOTICE/归属见 `ghostlock/README.md`。
- TIIEHenry 仓库本身**未声明许可证**；这里只镜像了其中的上游 GhostLock APK，没有复制它的 profile、脚本或文档。

校验（在仓库根目录执行）：

```bash
sha256sum -c SHA256SUMS
```

Windows PowerShell（同样在仓库根目录）：

```powershell
Get-Content .\SHA256SUMS | ForEach-Object {
  $h, $p = $_ -split '\s+', 2
  $a = (Get-FileHash -Algorithm SHA256 -LiteralPath $p).Hash.ToLower()
  if ($a -eq $h) { "OK   $p" } else { "FAIL $p" }
}
```

清单文件：[SHA256SUMS](../SHA256SUMS)。

返回 [README](../README.md)。