# DirtyFrag v1.17

- 文件：`DirtyFrag-v1.17.apk`
- 大小：`292,317` 字节
- SHA-256：`e02bd8e11433aa9676d9fb5d4bc7f42825d588015554d5a9e6fdae831a572337`
- 包名：`df.root`
- 上游：[mitschud/DirtyFrag](https://github.com/mitschud/DirtyFrag)
- 上游 tag：`v1.17`
- 许可证：**GPL-2.0**（见 `LICENSE-GPL-2.0.txt`）

本 APK 为上游 release 的**未修改**副本，没有重新签名或注入代码。

## GPL-2.0 源码声明

DirtyFrag v1.17 的完整对应源码可从上游获取：

- 仓库：<https://github.com/mitschud/DirtyFrag>
- tag：<https://github.com/mitschud/DirtyFrag/tree/v1.17>
- 上游是 [diabl0w/DFRoot](https://github.com/diabl0w/DFRoot)（GPL-2.0）的 fork。

如果你在分发本 APK，请保留 `LICENSE-GPL-2.0.txt` 和本声明，并提供上游源码链接。若上游链接不可用，请通过本仓库 issue 联系维护者。

## 可选内核模块

`module/dfroot-android13-5.15.ko`

- 大小：`18,528` 字节
- SHA-256：`50a790febd818e92886ecbd6a8e1230221eb1d1b85609f1ea5ee491151b7f8c1`
- 来源：[qq2070006042-create/yuehong-privilege-assistant](https://github.com/qq2070006042-create/yuehong-privilege-assistant)
- 许可证：**MPL-2.0**（见 `module/LICENSE-MPL-2.0.txt`）
- 说明：这是 5.15 系列的 dfroot 模块，vermagic 为 `5.15.202-android13-5.15.202_r00-dirty`。DirtyFrag APK 自身已经内置多版本模块；这个文件是可选的研究/兼容性参考，不是必须。

## 安装

完整步骤见 [docs/dirtyfrag.md](../../docs/dirtyfrag.md)。最短流程：

1. 校验 SHA-256。
2. Flyme 拦截时先放行包名：
   ```bash
   adb shell settings put secure usb_install_item_df.root 'DirtyFrag:1'
   ```
3. 安装 APK，打开，选 KernelSU，Run exploit → Are you sure。

> 只在你自己的设备上使用。APK 是 GPL-2.0 上游构建；本仓库不对其安全性或适用性作额外担保。

返回 [files/README.md](../README.md) 或 [主 README](../../README.md)。