# Lenovo ThinkStation P330 Tiny Hackintosh

OpenCore EFI for the Lenovo ThinkStation P330 Tiny running macOS Tahoe 26 (tested on 26.7.3, build 25G342), with **native macOS Wi-Fi and AirDrop** on a USB Realtek adapter.

---
## Configuration

| Specifications | Detail                                                  |
| ------------------- | ------------------------------------------- |
| Computer model | Lenovo ThinkStation P330 Tiny |
| Processor | Intel Core i7-9700T (Coffee Lake, 8C/8T, 2.0GHz) |
| Chipset | Intel Q370 |
| Memory | DDR4 SODIMM |
| Hard Disk | NVMe SSD |
| Integrated Graphics | Intel UHD Graphics 630 |
| Monitor | Arzopa Z1rc 16 inch (2560x1600@60hzs, HDMI) |
| Sound Card | Realtek ALC235 (layout-id: 11) |
| Ethernet | Intel I219-LM |
| Wireless Card | TP-Link Archer T2UB Nano (RTL8821CU, USB) |
| Bluetooth | RTL8821CU combo (WiFi + Bluetooth 4.2, USB) |
| Bootloader | OpenCore 1.0.8 |
| SMBIOS | iMac20,2 |

---
# What's working

| Feature | Status |
| ---------------- | ------------------------------------------- |
| GPU Acceleration | YES (UHD 630, QE/CI) |
| HDMI | YES |
| Monitor (Arzopa Z1rc 16 inch) | YES |
| Ethernet | YES (Intel I219-LM) |
| WiFi | YES — **native macOS Wi-Fi** (Wi-Fi menu, System Settings) on the Archer T2UB Nano (RTL8821CU): TahoeRealtekWiFi.kext + RTL8814AirLayer.kext |
| Bluetooth | YES (Archer T2UB Nano BT 4.2 — BlueToolFixup + RealtekBluetoothFirmware) |
| AirDrop | PARTIAL — iPhone ↔ Mac works at close range (about 10–50 cm), Mac on a **5 GHz** network; experimental, see [AirDrop](#airdrop) |
| Audio | YES (ALC235, layout-id 11) |


---
# Not tested

| Feature | Status |
| --------- | --------------------------------------------------- |
| DisplayPort (DP) | Not Tested |
| DRM | Software DRM works in Chrome-based browsers |

---
# Installation

### Identify your model

This EFI targets the **Lenovo ThinkStation P330 Tiny** with an **Intel Core i7-9700T (Coffee Lake)** and **Intel UHD Graphics 630**. Confirm your machine matches the Configuration table above before using this EFI.

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
8. **Generate your own SMBIOS** (serial, MLB, SmUUID, ROM) with GenSMBIOS and put it in `PlatformInfo > Generic`. The `config.plist` here ships with placeholder values (`W00000000001`, `M0000000000000001`); iMessage/FaceTime need your own.

### Upgrade

- Mount your EFI partition. Delete the existing `BOOT` and `OC` folders.
- Copy the `BOOT` and `OC` folders from the new EFI release.
- Keep your own `PlatformInfo` (SMBIOS) values in `config.plist`, then reboot.

---
# Notes

- **BIOS**: disable Secure Boot and CFG Lock, disable VT-d (or keep `DisableIoMapper`), set DVMT Pre-Allocated to 64MB+.
- **USB WiFi**: the TP-Link Archer T2UB Nano (RTL8821CU, `2357:013C` / `0BDA:C820`) works as **native macOS Wi-Fi** through `TahoeRealtekWiFi.kext` (radio driver) and `RTL8814AirLayer.kext` (makes macOS treat it like an Apple Wi-Fi card). Both load from OpenCore, so no app or installer is needed. The old Tahoe Wi-Fi menu-bar app is not used anymore.
  - Do **not** enable `IOSkywalkFamily`, `IO80211FamilyLegacy` or `AirPortBrcmNIC` (they stay disabled in this config): the AirLayer uses macOS's own Wi-Fi stack.
  - Boot-args used: `rtlair=2` (native Wi-Fi), `rtlawdl=1` (`awdl0`), `rtlairtrust=25G342`. The AirLayer only starts on macOS builds listed in its version gate or named by `rtlairtrust`; after a macOS update, set it to your new build (`sw_vers -buildVersion`).
- **Bluetooth**: the Archer T2UB Nano's BT 4.2 works with `BlueToolFixup` + `RealtekBluetoothFirmware`. The Broadcom kexts (`BrcmPatchRAM3`, `BrcmFirmwareRepo`) were removed; they are only for Broadcom Bluetooth. The config also resets `bluetoothExternalDongleFailed` and `bluetoothInternalControllerInfo` via NVRAM Add/Delete.
- **macOS Tahoe root patches**: use [laobamac/OCLP-Mod](https://github.com/laobamac/OCLP-Mod) for graphics/audio post-install patches on unsupported hardware.
  - OCLP-Mod downloads KDK/MetalLib files; Ethernet is the most reliable connection for that.

---
# AirDrop

AirDrop runs over AWDL on the USB adapter. It is **experimental**:

- Works: iPhone → Mac (including file transfer) at about 10–50 cm; Mac → iPhone (Finder) sometimes.
- Not reliable yet at 1 m or more (the adapter's radio from a rear USB port is weak; a USB extension cable helps).
- Connect the Mac's Wi-Fi to a **5 GHz** network (ideally channel 149, the AirDrop channel). On 2.4 GHz the radio would have to switch bands for AirDrop, which can drop Wi-Fi.
- On both devices set AirDrop to **Everyone for 10 Minutes** for testing.

This config ships with AirDrop transmit **on**:

```
rtlair=2 rtlawdl=1 rtlawdl_tx=1 rtlawdl_mcc=1 rtlawdl_ratemax=2 rtlairtrust=25G342
```

| Boot-arg | Effect |
| --- | --- |
| `rtlawdl_tx=1` | AWDL transmit (needed for AirDrop) |
| `rtlawdl_mcc=1` | serve AirDrop time slots while connected to Wi-Fi |
| `rtlawdl_ratemax=2` | cap AirDrop data at 24 Mb/s (more robust at a distance) |

To use native Wi-Fi without AirDrop, remove those three boot-args.

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
# Custom kexts

WiFi, AirDrop and Bluetooth on the TP-Link Archer T2UB Nano use custom-built kexts included in this EFI.

## TahoeRealtekWiFi.kext + RTL8814AirLayer.kext

- **Source**: [schorpy/TahoeRealtekWiFi, branch `unified-airlayer`](https://github.com/schorpy/TahoeRealtekWiFi/tree/unified-airlayer) — both kexts are built from that one repo (`./build.command --kext`, `cd airlayer && ./build_airlayer.sh`).
- **Location**: `EFI/OC/Kexts/TahoeRealtekWiFi.kext`, `EFI/OC/Kexts/RTL8814AirLayer.kext` (the AirLayer loads after the driver).
- The driver is [danielnmaldonado/TahoeRealtekWiFi](https://github.com/danielnmaldonado/TahoeRealtekWiFi) with a radio interface added; the AirLayer is [DrogaBox/RTL8814Native](https://github.com/DrogaBox/RTL8814Native)'s native IO80211 layer, attached to this driver.

Changes in the driver (see the fork's README for details):

- Radio interface (`rtlRadio*` calls) for the AirLayer: scan, join, data, raw 802.11 TX, AWDL receive hook, AWDL channel slots, AWDL port, delivery reports.
- AirDrop fixes: AWDL frames sent and labelled on the channel the radio is really on, hardware ACK port for `awdl0` with all unicast accepted while AWDL is active, recoverable channel switches, delivery reports reaching the AirLayer, rate fallback for AirDrop data.
- Native-Wi-Fi fixes: the private (randomized) MAC no longer breaks the chip identity check; scans while connected no longer drop the link.

The earlier local changes to build 67 (beacon-loss watchdog 2s → 30s in the RTL8192EU path, coex scan priority while unassociated) are **not** part of this build.

## RealtekBluetoothFirmware.kext (v1.0.7)

- **Location**: `EFI/OC/Kexts/RealtekBluetoothFirmware.kext`
- **Note**: `make` regenerates `Info.plist` — re-bump the version and re-add the `0bda:c820` personality after every rebuild.

Changes:

- Skip `SET_CONFIGURATION` during HCI init.
- Teardown fixes: `mInterruptResp` as a member (not stack), `mTerminating` flag, `terminateHci()` called from `willTerminate`.
- Result: no more kernel panics on replug with BT on.

---
# Credits

- [Acidanthera](https://github.com/acidanthera) — OpenCore, Lilu, VirtualSMC, WhateverGreen, AppleALC
- [chris1111](https://github.com/chris1111) — [Wireless USB OC Adapter drivers](https://github.com/chris1111/wireless-usb-big-sur-adapter) (RTL8821CU)
- [danielnmaldonado](https://github.com/danielnmaldonado) — [TahoeRealtekWiFi](https://github.com/danielnmaldonado/TahoeRealtekWiFi) (RTL8821CU WiFi) and Realtek Bluetooth firmware fixes
- [DrogaBox](https://github.com/DrogaBox) — [RTL8814Native](https://github.com/DrogaBox/RTL8814Native): the AirLayer (native macOS Wi-Fi + AWDL) and the reverse engineering behind it
- [laobamac](https://github.com/laobamac) — [OCLP-Mod](https://github.com/laobamac/OCLP-Mod) root patches
