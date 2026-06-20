# ASUS TUF Gaming X570-PLUS AMD Ryzen macOS Tahoe & Sequoia Hackintosh

English | [简体中文](Readme-zh.txt)

This project is based on OpenCore 1.0.7 and perfectly supports running **macOS Tahoe (15.x / 26.x)** and **macOS Sequoia (15.x)** on an **ASUS TUF Gaming X570-PLUS** motherboard with **AMD Ryzen 7 5800X3D** processor and **AMD Radeon RX 560** graphics card.

This configuration has been deeply adapted and optimized for macOS Sequoia and Tahoe, enabling standard desktop power management, onboard Realtek ALCS1200A audio, and legacy Broadcom BCM4360 Wi-Fi/Bluetooth via OCLP-Mod post-install root patching.

---

## 💻 Hardware Configuration and Support Status

| Hardware Components | Model | Driver Type | Status |
| :--- | :--- | :--- | :--- |
| **Processor (CPU)** | AMD Ryzen 7 5800X3D | `AMDRyzenCPUPowerManagement.kext` + `SMCAMDProcessor.kext` | Perfect Power Management |
| **Graphics Card (dGPU)** | AMD Radeon RX 560 (Polaris) | Native Support (`WhateverGreen.kext` helper) | Graphics Acceleration (Metal 3) |
| **Wired Network Adapter** | Realtek RTL8111 Gigabit Ethernet | `RealtekRTL8111.kext` | Normal |
| **Wireless Network Adapter** | Broadcom BCM4360 (PCIe / Fenvi) | OCLP-Mod (Restores legacy WiFi) + Kernel Injections | Normal (Airdrop/Handoff active) |
| **Bluetooth (BT)** | Broadcom Built-in Bluetooth | Native (mapped via USBMap.kext) | Normal |
| **Sound Card (Audio)** | Realtek ALCS1200A | Native HDA support + `AppleALC.kext` (Layout ID: 2) | Normal (Speakers, Headphones) |
| **Storage (NVMe)** | PCIe Gen4 NVMe SSD | Native OS Support + `NVMeFix.kext` | Normal |

---

## 🛠️ macOS Tahoe & Sequoia Special Configuration Notes (Must Read)

Since macOS Sequoia and Tahoe, Apple removed native drivers for legacy Broadcom wireless cards and legacy HDA components. To ensure all onboard components and legacy cards function flawlessly on these systems, this configuration implements critical security downgrade setups in `config.plist`:

1. **Disable Secure Boot:** `SecureBootModel` is set to **`Disabled`**.
   * *Reason:* Required for OCLP-Mod to apply kernel root patches and load unsigned drivers. Setting this to `x86legacy` or other values will cause boots to fail into Recovery mode.

2. **Relax System Sandbox & Signing:** `amfi=0x80` is added to `boot-args`.
   * *Reason:* Essential for loading legacy wireless drivers and sound patch extensions.

3. **Bypass Upgrade and Hardware Restrictions:** `RestrictEvents.kext` is loaded, and `revpatch=sb` is configured in `boot-args`.
   * *Reason:* Fakes secure boot to macOS updates servers, enabling official incremental over-the-air (OTA) updates when `SecureBootModel` is disabled.

---

## 🚀 Installation and Configuration Guide

### Step 1: Preparing EFI and Generating SMBIOS

1. Copy the entire `EFI` folder of this project to the EFI partition of your boot disk.
2. Use `GenSMBIOS` or `OCAuxiliaryTools` to generate unique serials for **`MacPro7,1`** or **`iMacPro1,1`** models.
3. Open `/EFI/OC/config.plist` and fill in your generated MLB, SystemSerialNumber, and SystemUUID under `PlatformInfo -> Generic`.

### Step 2: Motherboard BIOS Settings
For successful UEFI booting and native resolution display, set your motherboard BIOS as follows:
* **CSM (Compatibility Support Module):** `Disabled` (Crucial for widescreen output without black borders)
* **Above 4G Decoding:** `Enabled`
* **Fast Boot:** `Disabled` (Required for GPU handshake at boot to fix widescreen resolution)
* **Re-Size BAR Support:** `Disabled` (or `Auto` if required)
* **Secure Boot:** `Disabled` (Set to "Other OS")

### Step 3: macOS Installation & First Boot
1. Boot from your macOS installer USB drive and proceed with the standard macOS installation steps.
2. Upon first boot, you will notice that **WiFi is not working**. This is normal as Broadcom drivers need OCLP post-install patches. Please proceed to the next step using wired Ethernet or offline patching.

### Step 4: Run OCLP-Mod to Restore Broadcom Wireless (Crucial)
1. Download and open **OCLP-Mod (OpenCore Legacy Patcher Mod)**.
2. Click the **Post-Install Root Patch** button.
3. Click **Start Root Patching** and enter your macOS password. The tool will automatically rebuild the legacy Broadcom wireless driver kext cache.
4. **Restart your computer.** At the OpenCore picker menu, perform **"Reset NVRAM"** once to apply the new boot variables.
5. Your Broadcom BCM4360 WiFi and Bluetooth will now work flawlessly with native macOS controls!

---

## ⚙️ Personalized Tuning (Optional)

### 1. Correct CPU Display in About This Mac
The current configuration uses `revcpu=1` and `revcpuname=AMD Ryzen 7 5800X3D 8-Core Processor` in `boot-args` to ensure "About This Mac" displays the exact CPU model rather than "Unknown". You can modify `revcpuname` to match your actual CPU model.

### 2. Remove Verbose Mode (Clean Boot)
By default, `-v` (verbose mode) is enabled in `boot-args` for troubleshooting. For a clean, native Apple logo boot screen, open `/EFI/OC/config.plist` and remove `-v` from `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args`.

### 3. OpenCore Resolution Handshake Fix
We have set `UEFI -> Output -> ForceResolution = true` and recommended disabling `Fast Boot` in the BIOS. This completely resolves the stretched 4:3 display (with large black bars) on X570 boards during BIOS and OpenCore stages, giving you a beautiful full widescreen display.

### 4. Native Boot Chime Setup
We have fully configured the native pre-boot Mac sound chime at 70% volume:
- `AudioDxe.efi` is integrated into drivers.
- Lossless official `OCEFIAudio_VoiceOver_Boot.wav` is placed in `/EFI/OC/Resources/Audio/`.
- `AudioOutMask` is set to `-1` to output boot sound to both the rear green line-out and front-panel headphone jack.
- System volume is configured to recover automatically even after resetting NVRAM.

---

## 🤝 Acknowledgements
* [Acidanthera](https://github.com/acidanthera) for OpenCore Bootloader and various fundamental Kexts.
* [Dortania](https://github.com/dortania) for the outstanding OpenCore Install Guide.
* [OpenCore Legacy Patcher (OCLP-Mod) team](https://github.com/laobamac/OCLP-Mod) for the legacy Broadcom wireless driver restore solution on Sonoma/Sequoia/Tahoe.
