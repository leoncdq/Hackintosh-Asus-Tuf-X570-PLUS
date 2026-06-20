# AMD X570 OpenCore 引导配置文件 (适用于 macOS Tahoe & Sequoia)

专为 AMD Ryzen 锐龙处理器在 X570 芯片组主板平台定制、全面优化的高清全功能 OpenCore (v1.0.4+) 引导配置文件（EFI）。完美支持稳定引导 **macOS Tahoe (15.x)** 以及 **macOS Sequoia (15.x)**。

---

## 🌎 语言版本链接
* **中文说明文件 (Chinese Version)：** [Readme-zh.txt](./Readme-zh.txt)
* **英文说明文件 (English Version)：** [Readme.txt](./Readme.txt)

---

## 🖥️ 推荐目标硬件配置
* **主板：** AMD X570 芯片组系列主板 (AM4 插槽，如 华硕/微星/技嘉 等平台)
* **处理器：** AMD Ryzen 锐龙系列中央处理器 (AM4 系列)
* **集成声卡：** Realtek ALCS1200A 板载声卡芯片 (高清模拟音频)
* **无线网卡：** 博通 Broadcom BCM4360 系列无线网卡（经典免驱网卡，新系统需 OCLP Root 补丁）

---

## ✨ 核心优化与修复功能详情

### 1. 内核底层与引导优化
* **修正 `SetupVirtualMap` 为 `false`**：针对 AMD 平台的内存映射进行了最佳配置，彻底根治了类似 `StartImage failed - Aborted` 引起的开机引导卡死问题。
* **桌面端标准 ACPI 补丁**：内置了适用于 AMD 台式机的精简 SSDT 表（如 `SSDT-EC-USBX.aml`），确保 USB 供电与基础电源管理正常。

### 2. 博通 BCM4360 无线网卡全功能复活
针对 macOS Sequoia 及 Tahoe 彻底砍掉博通免驱网卡驱动的问题，本配置集成了完整的 OCLP 依赖链，实现无缝复苏：
* **屏蔽原生 Skywalk 驱动**：在内核阻止（Block）中排除了原生 `com.apple.iokit.IOSkywalkFamily`，避免系统驱动冲突。
* **按序注入遗留网卡驱动**：按严格的先后引导依赖顺序，依次加载注入 `IOSkywalkFamily.kext`、`IO80211FamilyLegacy.kext` 及子驱动 `AirPortBrcmNIC.kext`，完美还原底层驱动架构。
* **完美适配 OCLP Root 补丁**：注入了 `AMFIPass.kext` 并配置了 `amfi=0x80` 启动参数与相应的 NVRAM 解锁参数（`csr-active-config`），确保用户进入系统后可以流畅地打入 OCLP-Mod 的 Root Patch 补丁。

### 3. ALCS1200A 板载集成声卡驱动
* **声卡 Layout ID 注入**：在 `DeviceProperties` 中，为声卡物理路径 `PciRoot(0x0)/Pci(0x8,0x1)/Pci(0x0,0x4)` 精准注入了 `layout-id = 2`。
* **原生音频输出支持**：通过 `AppleALC.kext` 驱动该芯片，使输入与输出设备（扬声器、耳机）在系统内能开箱即用。

### 4. 极致精美的 OpenCanopy Widescreen 图形界面
* **HeiPG\Heikintosh 华丽主题**：默认启用美观奢华的 macOS 经典拟真图形启动菜单。
* **字体缺失报错完美修复**：彻底解决了因缺少字体资源导致的 `OCUI: Font init failed` 报错，完整补全了官方必备的 6 个字体二进制及配置文件（`/EFI/OC/Resources/Font/`）。
* **清理临时碎片**：清理了不必要的 macOS 属性碎片（如 `._` 文件），确保 FAT32 物理分区的 UEFI 极致兼容性。

### 5. 经典 Mac 开机叮～声音 (Boot Chime)
* **Pre-boot 阶段音频**：注入 `AudioDxe.efi` 驱动，并完整重构了 `UEFI -> Audio` 音频配置。
* **采用无损 PCM 声音**：删除了破损且不支持的 `.mp3` 格式音频，部署了官方原版无损 `OCEFIAudio_VoiceOver_Boot.wav` 音频文件。
* **前后置耳机音箱同时发声**：将 `AudioOutMask` 配置为 `-1`，使开机声音能同时发送至主板后置绿色音频孔以及机箱前置耳机插孔。
* **NVRAM 自动音量恢复**：在 `NVRAM -> Add` 模板中注入 `SystemAudioVolume` 键值 `<data>Rg==</data>`（十进制约 70 黄金开机音量），并加入了 `NVRAM -> Delete` 清除列表。即使电脑执行 “Reset NVRAM” 选项，开机叮声依然能自动刷新并恢复。

### 6. 开机 BIOS 与 OpenCore 菜单分辨率完美全屏
* **画面黑边故障解决**：修复了在开机阶段因显卡 VBIOS 握手不成功而导致主板 Logo 画面与 OpenCore 图形选单退缩至 4:3 比例、两侧带有宽大黑边的问题。
* **启用 `ForceResolution`**：将 `UEFI -> Output -> ForceResolution` 设置为 **`true`**，强行在引导加载阶段切换为显示器的最大物理分辨率。
* **BIOS 参数同步**：在主板 BIOS 中关闭 **`Fast Boot`** 选项，成功让显卡能够正常与显示器握手。现在，主板 Logo 以及 OpenCore 菜单均能完美、全屏、高清铺满输出。

---

## ⚙️ 推荐主板 BIOS 设置
为确保系统能顺利通过 UEFI 引导并正确握手全屏分辨率，请务必进行如下设置：
* **CSM (兼容性支持模块)：** `Disabled` (关闭 - 开启此项是导致分辨率降低和大黑边的主要成因)
* **Above 4G Decoding：** `Enabled` (开启)
* **Fast Boot (快速启动)：** `Disabled` (关闭 - 解决屏幕无信号、黑屏及分辨率不正常的关键)
* **Re-Size BAR Support：** `Disabled` (关闭，或设为 `Auto`)
* **Secure Boot (安全启动)：** `Disabled` (设为 “Other OS”)

---

## 🚀 使用方法说明
1. 将本项目目录下的 `EFI` 文件夹，直接复制到您系统启动盘的 EFI 分区中。
2. 使用专业的配置文件编辑器（如 OCAT 或 Xcode）打开 `/EFI/OC/config.plist`。
3. 进入 `PlatformInfo -> Generic` 选项，根据您的显卡和处理器重新生成一组 SMBIOS 序列号（推荐使用 `iMacPro1,1` 或 `MacPro7,1`）。
4. 保存配置，开机并在引导菜单中先执行一次 **"Reset NVRAM"** 重置选项，随后即可尽情体验高清无暇的 macOS 引导与运行环境！
