# 华硕 TUF Gaming X570-PLUS 锐龙 AMD Ryzen macOS Tahoe & Sequoia 黑苹果 EFI

[English](README.md) | 简体中文

本项目基于 OpenCore 1.0.7 引导，完美支持在 **ASUS TUF Gaming X570-PLUS** 主板、**AMD Ryzen 7 5800X3D** 处理器与 **AMD Radeon RX 560** 独立显卡平台上运行 **macOS Tahoe (15.x / 26.x)** 与 **macOS Sequoia (15.x)**。

本配置针对 macOS Sequoia 和 Tahoe 系统进行了深度的适配与调优，集成了原生电源管理、板载 Realtek ALCS1200A 声卡驱动，并通过 OCLP-Mod 免驱打补丁方式完美复活博通 BCM4360 无线网卡和蓝牙。

---

## 💻 硬件配置与支持状态

| 硬件组件 | 型号 | 驱动方式 | 状态 |
| :--- | :--- | :--- | :--- |
| **处理器 (CPU)** | AMD Ryzen 7 5800X3D | `AMDRyzenCPUPowerManagement.kext` + `SMCAMDProcessor.kext` | 完美电源管理与温度监控 |
| **显卡 (dGPU)** | AMD Radeon RX 560 (Polaris) | 免驱（`WhateverGreen.kext` 辅助） | 完美图形加速 (Metal 3) |
| **有线网卡** | Realtek RTL8111 Gigabit Ethernet | `RealtekRTL8111.kext` | 正常运行 |
| **无线网卡** | Broadcom BCM4360 (PCIe 系列 / Fenvi) | OCLP-Mod (补回遗留驱动) + 内核注入 | 正常运行 (支持 Airdrop/Handoff) |
| **蓝牙 (BT)** | Broadcom 内建蓝牙 | 原生支持 (通过 USBMap.kext 完美定制端口) | 正常运行 |
| **声卡 (Audio)** | Realtek ALCS1200A | `AppleALC.kext` (Layout ID: 2) | 原生驱动 (前后置插孔正常输出) |
| **存储 (NVMe)** | PCIe Gen4 NVMe 固态硬盘 | 原生支持 + `NVMeFix.kext` 功耗优化 | 正常运行 |

---

## 🛠️ macOS Tahoe & Sequoia 特殊配置说明 (必读)

自 macOS Sequoia 和 Tahoe 开始，苹果彻底删除了老旧 Mac 所使用的博通免驱无线网卡驱动与部分传统 HDA 音频组件。为了使硬件驱动在这些新系统上完美运行，本项目在 `config.plist` 中做了以下关键的安全降级配置：

1. **禁用安全启动**：`SecureBootModel` 被设为 **`Disabled`**。
   * *原因：这是使用 OCLP-Mod 写入内核驱动（Root Patch）且能正常开机引导的前提。若设为 `x86legacy` 或开启安全启动，会导致重启直接崩溃进入恢复模式。*

2. **放宽系统沙盒与签名**：`boot-args` 中加入了 **`amfi=0x80`**。
   * *原因：运行未签名的无线网卡和声卡补丁驱动所必需。*

3. **绕过升级与硬件检测限制**：使用了 `RestrictEvents.kext` 并配置了启动参数 **`revpatch=sb`**。
   * *原因：在 `SecureBootModel` 禁用的状态下伪装安全启动，使系统能正常接收官方小版本增量更新 (OTA) 推送。*

---

## 🚀 安装与配置指引

### 第一步：准备 EFI 与生成机型三码

1. 将本项目整个 `EFI` 文件夹拷贝至您的启动盘 EFI 分区中。
2. 使用 `GenSMBIOS` 或 `OCAuxiliaryTools` 生成适用于 **`MacPro7,1`** 或 **`iMacPro1,1`** 机型的三码（MLB, SystemSerialNumber, SystemUUID）。
3. 用文本编辑器或配置工具打开 `/EFI/OC/config.plist`，填入 `PlatformInfo -> Generic` 下对应的位置。

### 第二步：推荐主板 BIOS 设置
为确保系统能顺利通过 UEFI 引导并正确握手全屏分辨率，请务必进行如下设置：
* **CSM (兼容性支持模块)：** `Disabled` (关闭 - 开启此项是导致分辨率降低和大黑边的主要成因)
* **Above 4G Decoding：** `Enabled` (开启)
* **Fast Boot (快速启动)：** `Disabled` (关闭 - 解决屏幕无信号、黑屏及分辨率不正常的关键)
* **Re-Size BAR Support：** `Disabled` (关闭，或设为 `Auto`)
* **Secure Boot (安全启动)：** `Disabled` (设为 “Other OS”)

### 第三步：安装 macOS 系统
1. 按照标准的黑苹果安装步骤通过 USB 启动盘安装系统。
2. 首次进入系统后，您会发现 **WiFi 无法工作**，这是正常现象（因为博通驱动尚未注入）。请使用有线网卡或离线方式进行下一步。

### 第四步：运行 OCLP-Mod 恢复博通无线网卡 (关键步骤)
1. 下载并打开 **OCLP-Mod (OpenCore Legacy Patcher Mod)**。
2. 点击 **Post-Install Root Patch** 按钮。
3. 点击 **Start Root Patching** 并输入您的开机密码。软件将自动下载并补回博通无线网卡的系统级遗留驱动。
4. **重启电脑**。在进入 OpenCore 引导菜单时，**务必执行一次 "Reset NVRAM" 重置**，随后引导进入系统。
5. 此时进入系统后，您的博通 BCM4360 无线网卡与蓝牙将完美复活，支持原生 macOS Wi-Fi 控制和空投功能！

---

## ⚙️ 个性化调优 (可选)

### 1. 修正“关于本机”中的 CPU 显示
当前配置在 `boot-args` 中集成了 `revcpu=1` 和 `revcpuname=AMD Ryzen 7 5800X3D 8-Core Processor` 参数。这保证了在“关于本机”界面中能精准显示您的 CPU 完整型号，而不是显示 "Unknown" (未知)。您可以根据实际 CPU 型号自行更改 `revcpuname`。

### 2. 关闭 Verbose 跑码模式 (纯净开机)
默认情况下，为了便于排查问题，我们在 `boot-args` 中开启了 `-v` (跑码模式)。如果您想享受白苹果 Logo 的优雅开机画面：
- 使用编辑器打开 `/EFI/OC/config.plist`，从 `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args` 中删除 **`-v`** 即可。

### 3. 开机 BIOS 与 OpenCore 选单分辨率完美全屏
通过设置 `UEFI -> Output -> ForceResolution = true` 并在主板 BIOS 中关闭 `Fast Boot`。这成功解决了在开机阶段因显卡 VBIOS 握手问题导致 OpenCore 图形选单退缩至 4:3 比例、两侧带有宽大黑边的通病，开机菜单现在可以完美高清全屏铺满。

### 4. 经典 Mac 开机 “叮～” 声音 (Boot Chime)
我们已为您完全配置好 70% 黄金音量的开机声音：
- 集成 `AudioDxe.efi` 驱动。
- 部署官方无损 `OCEFIAudio_VoiceOver_Boot.wav` 于 `/EFI/OC/Resources/Audio/`。
- 将 `AudioOutMask` 设置为 `-1`，使开机声音能同时发送至主板后置绿色音频孔与机箱前置耳机插孔。
- 实现了 NVRAM 音量记忆。即使执行 "Reset NVRAM"，开机声音也能自动刷新并完美保留。

---

## 🤝 致谢
* [Acidanthera](https://github.com/acidanthera) 提供 OpenCore 引导器及核心 Kext 驱动。
* [Dortania](https://github.com/dortania) 提供行业标杆的 OpenCore 安装教程。
* [OpenCore Legacy Patcher (OCLP-Mod) 团队](https://github.com/laobamac/OCLP-Mod) 解决 Sonoma/Sequoia/Tahoe 上的博通免驱网卡完美复活方案。
