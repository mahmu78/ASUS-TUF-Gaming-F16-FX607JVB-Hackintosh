# ASUS TUF Gaming F16 FX607JVB Hackintosh

**Intel Core 5 210H • RTX 3050 Laptop GPU • macOS Sequoia**

⚠️ **Daily usable, but not fully stable.**

A documented, community-maintained OpenCore EFI for the ASUS TUF Gaming F16 FX607VJB (Intel Core 5 210H, RTX 3050 6GB Laptop GPU). macOS Sequoia runs with E-cores enabled. In Standard (hybrid) graphics mode the main acceleration comes from the Intel iGPU through WhateverXe 0.8.5 (Metal 2); in Ultimate (dGPU only) mode the NVIDIA dGPU is accelerated through the NullMoth driver. The machine dual-boots with Windows 11 on a single NVMe SSD.

**Model note:** the Core 5 210H variant is sold as FX607VJB (for example FX607VJB-RL136). This repository was previously titled "FX607JVB"; both spellings are kept here so the project is easy to find.

This is a personal, device-specific build. It was tested on one laptop only. Other revisions, BIOS versions or storage layouts may behave differently. Do not copy it blindly; compare it with your own config.

## Security notice

All unique platform identifiers (SystemSerialNumber, MLB, SystemUUID, ROM) in `config.plist` are cleared. Generate your own values with GenSMBIOS before booting. The SMBIOS model used in this config is `MacBookPro16,2`.

No Apple software is distributed in this repository, only the EFI folder.

## Tested versions

| Component | Version |
|---|---|
| macOS | Sequoia 15.8.3 (24H241), also tested on 15.8.1 |
| OpenCore | 1.0.8 |
| NullMoth (NVIDIA driver) | 3.0 |
| WhateverXe / WhateverXe Graphics | 0.8.5 / 0.4.5 (iGPU working, see below) |
| AsusBridge | 3.0 |
| RTL8111 (Ethernet) | 2.4.2, see "Ethernet" |
| AirPort_RTW89 (Wi-Fi) | 0.6.0 |
| BIOS | FX607VJB.338 |

## System specifications

| Component | Specification |
|---|---|
| Laptop | ASUS TUF Gaming F16 FX607VJB |
| CPU | Intel Core 5 210H (hybrid: 4 P-cores + 4 E-cores), E-cores enabled; shown as "Intel Core 5 210H" in About This Mac |
| dGPU | NVIDIA GeForce RTX 3050 6GB Laptop GPU (NullMoth) |
| iGPU | Intel Raptor Lake-P graphics, PCI ID 8086:A7AB, working with WhateverXe 0.8.5 (1536 MB, Metal 2); main accelerator in Standard mode |
| RAM | 8GB DDR5 5600MHz |
| Storage | 512GB Western Digital NVMe SSD (single-disk dual boot with Windows 11) |
| Wi-Fi / Bluetooth | Realtek RTL8852BE |
| Ethernet | Realtek Gigabit Ethernet (RTL8111.kext worked) |

## What works

- **dGPU:** RTX 3050 with Metal 3 hardware acceleration via NullMoth in Ultimate (dGPU only) mode [state whether it also works in Standard mode].
- **iGPU:** Intel Graphics (8086:A7AB) with 1536 MB and Metal 2 through WhateverXe 0.8.5 + WhateverXe Graphics 0.4.5. In Standard (hybrid) mode it is the main accelerator and drives the built-in display.
- **Built-in display:** full resolution and refresh rate through dGPU routing in Ultimate mode; in Standard mode the display is driven by the iGPU.
- **Audio:** built-in speakers and 3.5mm headphone jack (AppleALC).
- **Wi-Fi and Bluetooth:** RTL8852BE works on home networks (WPA2/WPA3) and on enterprise networks (tested on a university network and a dormitory network) with AirPort_RTW89 0.6.0. Bluetooth audio and peripherals work.
- **Ethernet:** works with the kext version listed above (see "Ethernet").
- **Power and thermals:** CPU power management, battery readout, fan telemetry and an 80% charge limit through AsusBridge.
- **Input:** keyboard (including backlight and Fn hotkeys) and touchpad with multi-touch gestures.
- **USB and storage:** all USB-A / USB-C ports, NVMe SSD at full speed.
- **Webcam:** works in Safari.

## BIOS settings

| Setting | Value |
|---|---|
| Secure Boot | Disabled |
| Fast Boot | Disabled |
| CSM | Disabled (required by NullMoth) |
| Above 4G Decoding | Enabled (required by NullMoth) |
| Graphics mode (BIOS / Armoury Crate) | Standard (hybrid) for the iGPU (main accelerator); Ultimate / dGPU only for NullMoth |
| E-cores | Enabled |
| VMD Controller | Disabled (or AHCI mode if supported) |
| CFG Lock / VT-d | Handled through OpenCore quirks |

**Note:** use Standard mode for the iGPU and Ultimate (dGPU only) mode for the NullMoth dGPU. The Ultimate mode may be available in the BIOS; if it is not, set it in Windows with Armoury Crate. Option names differ between BIOS versions, so check these settings against your own BIOS before booting.

## E-cores enabled: the EXITBS:START freeze

With E-cores enabled in BIOS, OpenCore stopped at `EXITBS:START` and macOS never started. With E-cores disabled it booted normally.

Things that did not change anything: `cpus=1`, disabling `CpuTopologyRebuild.kext`, disabling the force-HT kernel patch, disabling `APIC.aml`, switching the GPU mode between dGPU-only and Standard.

What fixed it: a small change in the CPUID masking.

```
Kernel -> Emulate
  Before: Cpuid1Data = 55060A00000000000000000000000000
          Cpuid1Mask = FFFFFFFF000000000000000000000000
  After:  Cpuid1Data = 55060A00000080000000000000000000
          Cpuid1Mask = FFFFFFFF0000FF000000000000000000
```

## Ethernet (RTL8111)

The latest `RealtekRTL8111.kext` release I downloaded (version 3.0) did not work on macOS 15.8.3 (symptom: no link). The version bundled by OpCore Simplify (2.4.2) works.

## Benchmarks (Geekbench 7.1.0)

All tests were run on AC power with Armoury Crate set to Ultimate (dGPU) mode. The GPU rows are for the NVIDIA dGPU only; no iGPU benchmark result is available (see Known limitations).

| Test | Windows 11 | macOS Sequoia 15.8.3 (NullMoth) | macOS vs Windows |
|---|---|---|---|
| CPU, single-core | 1,854 | 971 | 52% |
| CPU, multi-core | 8,154 | 5,643 | 69% |
| GPU, Vulkan (Windows) / Metal (macOS) | 46,827 | 45,135 | 96% (about 3.6% lower) |
| GPU, OpenCL | 48,652 | 26,984 | 55% |

Notes: macOS does not schedule P-cores and E-cores as efficiently as Windows, which lowers CPU scores. OpenCL is deprecated on macOS and runs through a translation layer.

Screenshots (About This Mac, CPU and GPU comparisons) are in the `screenshots/` folder.

## Known limitations

- **Intel iGPU:** history and limits. With WhateverXe 0.8.2 / 0.8.3 and Graphics 0.4.4 the iGPU (8086:A7AB) was admitted but the display hand-off never completed, so there was no acceleration. WhateverXe 0.8.5 with Graphics 0.4.5 fixed it on this laptop, and in Standard mode the iGPU is now the main accelerator (Metal 2). Upstream still lists Raptor Lake-H/P internal eDP as not tested on hardware, so treat this as one working data point.
- **iGPU compute benchmark:** running the Geekbench GPU test on the Intel iGPU freezes. Heavy compute loads on the iGPU are not verified.
- **Deep sleep / hibernation:** entering deep sleep causes a kernel panic and a reboot (PCIe power-state mismatch under NullMoth). Use normal screen lock / display sleep.
- **NVIDIA HDMI / DisplayPort audio:** audio over external displays through the dGPU does not work.
- **Heavy Java mods (for example Minecraft MTR, Create):** occasional stutter, likely from P-core / E-core scheduling.
- **Browsers:** the camera works in Safari. Chromium-based browsers apply stricter sandbox / TCC limits. Intermittent rendering glitches and video stutters occur on VP9 and AV1 streams due to hardware video decode / Metal synchronization behavior. Forcing H.264 playback (e.g., via `h264ify` / `enhanced-h264ify`) resolves the stutter and ensures smooth playback.
- **UI artifacts:** About This Mac or System Settings may occasionally render blank or show dark text on light backgrounds [re-test with the iGPU working and update this line].
- **Battery charge limit in dual boot:** the 80% limit is handled by the ASUS embedded controller, so switching between Windows and macOS can occasionally reset it.

## Using AsusBridge

On modern ASUS laptops I use AsusBridge instead of AsusSMC because it provides working battery charge limits, fan telemetry and ASUS-specific controls on this model.

## License

The configuration files in this repository are released under the MIT License. The kexts, drivers and tools referenced below keep their own licenses.

## Acknowledgments

- **AsusBridge (developed by ixboss):** ASUS EC telemetry, keyboard backlight and battery charge limit.
- **Airport_RTW89 (developed by FranciumOS-org):** Realtek RTL8852BE Wi-Fi on macOS.
- **NullMoth Nvidia Driver:** Metal 3 acceleration for the RTX 3050.
- **RTL8111.kext (developed by mieze):** Realtek gigabit ethernet on macOS.
- **WhateverXe** by b00t0x: Intel Xe iGPU work and CpuTopologyRebuild for hybrid CPUs.
- **Acidanthera:** OpenCore, Lilu, VirtualSMC and other core kexts.
- **Dortania:** OpenCore Install Guide.
- **OpCore Simplify:** initial EFI generation.
