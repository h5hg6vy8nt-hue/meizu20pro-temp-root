# 开源、引用与许可证

本仓库是一份**文档型**指南，不是 exploit 发行版。

- 不包含漏洞利用源码；
- 不镜像 APK；
- 不附带内核模块；
- 所有工具、二进制、profile 请从上游项目获取。

---

## 1. 本仓库的许可

除特别注明外，本仓库中的文字、表格、命令示例采用 **Creative Commons Attribution-ShareAlike 4.0 International（CC BY-SA 4.0）**。

你可以：

- 复制、转载、翻译、改编；
- 用于商业或非商业目的。

但必须：

- 注明出处（本仓库链接）；
- 保留作者署名；
- 以相同许可（CC BY-SA 4.0）发布衍生作品；
- 不附加额外法律限制。

完整许可文本见 [LICENSE](../LICENSE)，或：

<https://creativecommons.org/licenses/by-sa/4.0/>

---

## 2. 引用方式

文字引用建议：

> 魅族 20 Pro 临时 Root 指南（锁定 BL），DirtyFrag / GhostLock 实机记录，<仓库 URL>，访问日期 YYYY-MM-DD。

学术/文章引用可使用：

```bibtex
@misc{meizu20pro_temp_root_guide,
  title  = {魅族 20 Pro 临时 Root 指南（锁定 BL）},
  year   = {2026},
  note   = {DirtyFrag (CVE-2026-43284) and GhostLock (CVE-2026-43499) on MEIZU 20 Pro with locked bootloader},
  url    = {https://github.com/h5hg6vy8nt-hue/meizu20pro-temp-root}
}
```

---

## 3. 上游项目与许可证

| 项目 | 在本指南中的用途 | 许可证 | 是否再分发 |
| --- | --- | --- | --- |
| [mitschud/DirtyFrag](https://github.com/mitschud/DirtyFrag) | DirtyFrag APK / `dfroot.ko` | GPL-2.0 | 否，仅链接 |
| [diabl0w/DFRoot](https://github.com/diabl0w/DFRoot) | DirtyFrag 上游 | GPL-2.0 | 否，仅链接 |
| [yukonga/ghostlock-app](https://github.com/yukonga/ghostlock-app) | GhostLock APK | Apache-2.0 | 否，仅链接 |
| [TIIEHenry/Meizu20ProRoot](https://github.com/TIIEHenry/Meizu20ProRoot) | 魅族 20 Pro GhostLock APK/profile/文档 | **未声明许可证** | 否，仅链接 |
| [qq2070006042-create/yuehong-privilege-assistant](https://github.com/qq2070006042-create/yuehong-privilege-assistant) | 5.15 dfroot 模块参考 | MPL-2.0 | 否，仅链接 |
| [tiann/KernelSU](https://github.com/tiann/KernelSU) | Root 后端 | GPL-3.0 | 否，仅链接 |
| [RikkaApps/Shizuku](https://github.com/RikkaApps/Shizuku) | GhostLock 的 shell 入口 | Apache-2.0 | 否，仅链接 |
| [tytydraco/LADB](https://github.com/tytydraco/LADB) | 纯手机端 adb 备选 | 见仓库（NOASSERTION） | 否，仅链接 |

### 关于 TIIEHenry/Meizu20ProRoot

该仓库**没有 LICENSE 文件**，按默认版权规则保留所有权利。本指南：

- 只链接到该仓库；
- 不复制、不镜像、不再分发其 APK、profile、脚本或文档；
- 文中提到的偏移值、测试结果属于本机实测事实，不是对其文件的再分发。

如果你要使用/转载该仓库的文件，请自行联系作者确认授权。

---

## 4. 漏洞公告

| CVE | 名称/说明 | 链接 |
| --- | --- | --- |
| CVE-2026-43284 | Dirty Frag：IPsec ESP 页面缓存写 | <https://nvd.nist.gov/vuln/detail/CVE-2026-43284> |
| CVE-2026-43500 | Dirty Frag 关联变体（RxRPC） | <https://nvd.nist.gov/vuln/detail/CVE-2026-43500> |
| CVE-2026-46300 | Fragnesia | <https://nvd.nist.gov/vuln/detail/CVE-2026-46300> |
| CVE-2026-43499 | GhostLock 使用的 futex/rt_mutex 漏洞 | <https://nvd.nist.gov/vuln/detail/CVE-2026-43499> |

漏洞细节、影响范围和修复状态以 NVD、MITRE、内核公告和厂商公告为准。本指南只记录在特定设备上的可用性，不对漏洞本身作权威描述。

---

## 5. 事实来源与验证

本指南中的结论分三类：

1. **实机观察**：在 MEIZU 20 Pro（`5.15.119-android13-8-gdae8b7f03305-ab1764145455`）上实际执行、读日志、重复验证得到的结果；
2. **上游文档**：APK/仓库 README、release notes、源码中的字符串与行为；
3. **静态分析**：对 APK、DEX、`libdfroot.so`、`dfroot.ko` 的解包与字符串分析。

没有实机验证的内容会明确标注“未验证/不保证”。不同机型、内核、Flyme 版本请自行评估风险。

---

## 6. 商标与归属

- Android 是 Google LLC 的商标；
- MEIZU、Flyme 是魅族科技的商标；
- KernelSU、Shizuku、LADB、DirtyFrag、DFRoot、GhostLock 等名称归各自作者/项目所有；
- 本指南与上述公司/项目无隶属或背书关系。

---

## 7. 致谢

感谢：

- `mitschud` 与 `diabl0w`：DirtyFrag / DFRoot；
- `yukonga`：GhostLock；
- `TIIEHenry`：魅族 20 Pro 的 GhostLock 适配与实机记录；
- `qq2070006042-create`：5.15 dfroot 模块参考；
- KernelSU、Shizuku、LADB 的维护者；
- 以及所有公开披露 CVE 细节的安全研究者。

没有这些公开工作，这份文档不会存在。

---

## 8. 免责声明

本指南仅用于你自己的设备、安全研究与学习。内核漏洞利用存在数据丢失、系统损坏、保修失效的风险；作者不对任何直接或间接后果负责。请遵守当地法律法规与设备厂商条款。

返回 [README](../README.md)。