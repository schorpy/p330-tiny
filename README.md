# Lenovo ThinkCentre P330 Tiny Hackintosh

OpenCore EFI for the Lenovo ThinkCentre P330 Tiny running macOS Tahoe 26.5

---
## Configuration

| Specifications | Detail                                                  |
| ------------------- | ------------------------------------------- |
| Computer model | Lenovo ThinkCentre P330 Tiny |
| Processor | Intel Core i7-9700T (Coffee Lake, 8C/8T, 2.0GHz) |
| Chipset | Intel Q370 |
| Memory | DDR4 SODIMM |
| Hard Disk | NVMe SSD |
| Integrated Graphics | Intel UHD Graphics 630 |
| Monitor | Arzopa Z1rc 16 inch (2560x1600@60hzs, HDMI) |
| Sound Card | Realtek ALC235 (layout-id: 11) |
| Ethernet | Intel I219-LM |
| Wireless Card | TP-Link Archer T2UB Nano (RTL8821CU, USB) / MTech UW-01 802.11N USB 2.0 (RTL8188FTV) |
| Bluetooth | RTL8821CU combo (USB, not usable in macOS) |
| Bootloader | OpenCore 1.0.7 |
| SMBIOS | iMac20,2 |

---
# What's working

| Feature | Status |
| ---------------- | ------------------------------------------- |
| GPU Acceleration | YES (UHD 630, QE/CI) |
| HDMI | YES |
| Monitor (Arzopa Z1rc 16 inch) | YES |
| Ethernet | YES (Intel I219-LM) |
| WiFi | YES (TP-Link Archer T2UB Nano, RTL8821CU — [driver](https://github.com/chris1111/wireless-usb-big-sur-adapter)) |
| WiFi (alt) | YES (MTech UW-01 802.11N USB 2.0, RTL8188FTV) |
| Audio | YES (ALC235, layout-id 11) |


---
# Not tested

| Feature | Status |
| --------- | --------------------------------------------------- |
| DisplayPort (DP) | Not Tested |
| Bluetooth | Not usable (Realtek USB BT unsupported) |
| AirDrop | Not Tested (requires supported WiFi/BT) |
| DRM | Software DRM works in Chrome-based browsers |

---
# Installation

### Identify your model

This EFI targets the **Lenovo ThinkCentre P330 Tiny** with an **Intel Core i7-9700T (Coffee Lake)** and **Intel UHD Graphics 630**. Confirm your machine matches the Configuration table above before using this EFI.

### BIOS settings

- Disable **Secure Boot**
- Disable **CFG Lock**
- Disable **VT-d** (or keep `DisableIoMapper` enabled)
- Set **DVMT Pre-Allocated** to **64MB** or higher
- Set **SATA** mode to **AHCI**
- Enable **UEFI** boot, disable **CSM / Legacy**

### First-time installation

1. Download macOS and create a bootable USB installer (16GB+).
2. Mount the USB's EFI partition and copy the `EFI` folder (`BOOT` and `OC`) to the root of that partition.
3. Reboot and press `F12` to open the boot menu, then select the USB drive.
4. In OpenCore, choose **Reset NVRAM**, reboot, then select **Install macOS**.
5. In Disk Utility, erase the target disk as **APFS** with a **GUID Partition Map**.
6. Complete the macOS installation and boot into the system.
7. Mount the internal drive's EFI partition and copy the same `EFI` folder there so the system boots without the USB.
8. **Generate your own SMBIOS** (serial, MLB, SmUUID, ROM) with GenSMBIOS — never reuse the serials from this repo for iServices.

### Upgrade

- Mount your EFI partition. Delete the existing `BOOT` and `OC` folders.
- Copy the `BOOT` and `OC` folders from the new EFI release.
- Keep your own `PlatformInfo` (SMBIOS) values in `config.plist`, then reboot.

---
# Notes

- **BIOS**: disable Secure Boot and CFG Lock, disable VT-d (or keep `DisableIoMapper`), set DVMT Pre-Allocated to 64MB+.
- **USB WiFi**: use [chris1111/Wireless-USB-Big-Sur-Adapter](https://github.com/chris1111/wireless-usb-big-sur-adapter) for the TP-Link Archer T2UB Nano (RTL8821CU). The MTech UW-01 802.11N USB 2.0 (RTL8188FTV, `0BDA:F179`) also works with the same driver.
  - **Important**: connect to the internet via Ethernet cable first. USB WiFi is not available until its driver is installed, so Ethernet is required to download the driver.
- **macOS Tahoe root patches**: use [laobamac/OCLP-Mod](https://github.com/laobamac/OCLP-Mod) for graphics/audio post-install patches on unsupported hardware.
  - **Important**: connect to the internet via Ethernet cable first, since OCLP-Mod must download KDK/MetalLib files before USB WiFi is working.

---
# iPhone USB pairing

If macOS refuses to pair or trust a connected iPhone (no "Allow" prompt, device not visible in Finder), set SIP correctly:

- Set `csr-active-config` to `03000000` (Data: `AwAAAA==`).
  - OpenCore path: `NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82 > csr-active-config`
  - Make sure `csr-active-config` is also listed in `NVRAM > Delete` so the new value is applied on the next boot.
- `SecureBootModel` can stay `Disabled`.
- Reboot, connect the iPhone, and tap **Trust** when prompted.
- If pairing breaks after a macOS/OCLP root-patch session, check `nvram csr-active-config` first — it may have been rewritten.

---
# Credits

- [Acidanthera](https://github.com/acidanthera) — OpenCore, Lilu, VirtualSMC, WhateverGreen, AppleALC
- [chris1111](https://github.com/chris1111) — [Wireless USB OC Adapter drivers](https://github.com/chris1111/wireless-usb-big-sur-adapter) (RTL8821CU)
- [laobamac](https://github.com/laobamac) — [OCLP-Mod](https://github.com/laobamac/OCLP-Mod) root patches
