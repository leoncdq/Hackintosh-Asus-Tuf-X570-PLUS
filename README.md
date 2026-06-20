# Hackintosh-Asus-Tuf-X570-PLUS
Hackintosh Asus Tuf X570-PLUS OC1.0.7 Tahoe 26.5.1
# AMD X570 OpenCore EFI Configuration for macOS Tahoe & Sequoia

An optimized, full-featured OpenCore (v1.0.4+) EFI configuration tailored for AMD Ryzen processors on X570 platform motherboards, fully supporting **macOS Tahoe (26.5.x)** and **macOS Sequoia (15.x)**.

---

## 🌎 Language Links
* [中文说明](./Readme-zh.txt)
* [English](./Readme.txt)

---

## 🖥️ Target Hardware Specifications
* **Motherboard:** AMD X570 Chipset (Socket AM4, e.g., ASUS/Gigabyte/MSI platforms)
* **Processor:** AMD Ryzen CPUs (AM4 Series)
* **Onboard Audio:** Realtek ALCS1200A (HDA-compliant onboard codec)
* **Wireless & Bluetooth:** Broadcom BCM4360 (Apple legacy native card, e.g., Fenvi T919)

---

## ✨ Features & Optimizations

### 1. Core Loader & AMD Setup
* **`SetupVirtualMap` Correction:** Correctly set to `false` for optimal AMD memory mapping, preventing typical boot halts like `StartImage failed - Aborted`.
* **Standard ACPI Patches:** Includes essential desktop SSDTs (e.g., `SSDT-EC-USBX.aml`) for proper power management and USB mapping.

### 2. Broadcom BCM4360 Wi-Fi Restore
Since macOS Sequoia dropped native drivers for legacy Broadcom cards, we implemented the full OCLP dependency chain to bring it back seamlessly:
* **Skywalk Blockage:** Excludes native `com.apple.iokit.IOSkywalkFamily` in Kernel Block to prevent system driver conflicts.
* **Legacy Driver Injection:** Cleanly injects `IOSkywalkFamily.kext`, `IO80211FamilyLegacy.kext`, and `AirPortBrcmNIC.kext` in the exact required boot order.
* **Root Patch Preparation:** Injected `AMFIPass.kext` and configured `amfi=0x80` in `boot-args` along with customized `csr-active-config` in NVRAM, ensuring OCLP-Mod root patching can be executed and applied smoothly in Tahoe/Sequoia.

### 3. Onboard Audio ALCS1200A
* **Layout ID Injection:** Handled through `DeviceProperties` with `layout-id = 2` mapped to `PciRoot(0x0)/Pci(0x8,0x1)/Pci(0x0,0x4)`.
* **AppleALC Support:** Uses `AppleALC.kext` to fully drive the physical audio controller, offering rich native audio output out of the box.

### 4. OpenCanopy Widescreen Graphical UI Theme
* **HeiPG\Heikintosh Theme:** Fully configured beautiful macOS-style boot picker.
* **Font Issues Fixed:** Solved the common `OCUI: Font init failed` error by supplying all 6 official OpenCore font binaries (`Font_1x.bin`, `Font_1x.png`, `Font_2x.bin`, `Font_2x.png`, `Terminus.hex`, `TerminusCore.hex`) in `/EFI/OC/Resources/Font/`.
* **Clean Partition Meta:** Removed temporary system files (e.g., `._` files) for pure FAT32 BIOS compatibility.

### 5. Classical Mac Boot Chime Sound
* **Pre-boot Audio:** Injected `AudioDxe.efi` and fully configured the `UEFI -> Audio` dictionary block.
* **Lossless PCM Audio:** Cleaned up unsupported `.mp3` files and deployed the official lossless `OCEFIAudio_VoiceOver_Boot.wav` sound under `Resources/Audio/`.
* **Dual Output Support:** Configured `AudioOutMask` to `-1` to enable simultaneous audio playback over rear green line-out and front-panel headphone jacks.
* **NVRAM Persistence:** Injected `SystemAudioVolume` data `<data>Rg==</data>` (approx. 70% golden volume level) under `NVRAM -> Add` and added it to the `NVRAM -> Delete` block. This guarantees the boot chime is immediately audible and automatically recovers even after performing a "Reset NVRAM" option.

### 6. Widescreen Full-Screen Boot Resolution Fix
* **The Resolution Bug:** Fixed the issue where early boot (motherboard BIOS phase) and the OpenCore picker displayed inside a distorted 4:3 letterbox with wide black bars on both sides.
* **`ForceResolution` Enable:** Set `UEFI -> Output -> ForceResolution` to `true` to force a correct resolution transition from the motherboard firmware to the display's native maximum.
* **BIOS Sync:** Recommended disabling `Fast Boot` in BIOS to allow proper display handshakes during pre-boot. Both BIOS logo and OpenCore now scale to full widescreen.

---

## ⚙️ Recommended Motherboard BIOS Settings
For successful booting and native resolution display, set your motherboard BIOS as follows:
* **CSM (Compatibility Support Module):** `Disabled` (Crucial for UEFI GOP widescreen output)
* **Above 4G Decoding:** `Enabled`
* **Fast Boot:** `Disabled` (Crucial for resolving black screens / low resolution boot display)
* **Re-Size BAR Support:** `Disabled` (or `Auto` if required)
* **Secure Boot:** `Disabled` (Set to "Other OS")

---

## 🚀 How to Use
1. Copy the `EFI` directory from this repository directly to the EFI partition of your macOS boot drive.
2. Open `/EFI/OC/config.plist` using proper plist editors (e.g., OCAT or Xcode).
3. Under `PlatformInfo -> Generic`, generate a new SMBIOS appropriate for your setup (Recommended: `iMacPro1,1` or `MacPro7,1`).
4. Boot, select **"Reset NVRAM"** from the picker once, and enjoy the perfect macOS installation!
