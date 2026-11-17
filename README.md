# ASUS TUF Gaming F16 (FX607VJB, Raptor Lake-H) Hackintosh EFI

![macOS Sequoia](https://img.shields.io/badge/macOS-Sequoia%2015.8.3-blue) ![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8-green) ![License](https://img.shields.io/badge/License-MIT-lightgrey)

A documented, proof-of-concept OpenCore EFI for the **ASUS TUF Gaming F16 FX607VJB** (Intel Core 5 210H, RTX 3050 6GB Laptop GPU). macOS Sequoia runs with **E-cores enabled**, and the NVIDIA dGPU is accelerated through the **NullMoth** driver. The machine dual-boots with Windows 11 on a single NVMe SSD.

> **Model note:** the Core 5 210H variant is sold as **FX607VJB** (for example FX607VJB-RL136). This repository was previously titled "FX607JVB"; both spellings are kept here so the project is easy to find.

> **This is a personal, device-specific build.** It was tested on one laptop only. Other revisions, BIOS versions or storage layouts may behave differently. Do not copy it blindly; compare it with your own config.

## Security notice

All unique platform identifiers (`SystemSerialNumber`, `MLB`, `SystemUUID`, `ROM`) in `config.plist` are cleared. Generate your own values with GenSMBIOS before booting. The SMBIOS model used in this config is **[exact model name as written in config.plist]**.

No Apple software is distributed in this repository, only the EFI folder.

## Tested versions

| Component | Version |
| --- | --- |
| macOS | Sequoia 15.8.3 (24H241), also tested on 15.8.1 |
| OpenCore | 1.0.8 |
| NullMoth (NVIDIA driver) | [version] |
| WhateverXe / WhateverXe Graphics | 0.8.2 / 0.4.4 (iGPU not working, see below) |
| AsusBridge | [version] |
| RealtekRTL8111 (Ethernet) | [working version], see "Ethernet" |
| BIOS | [version] |

## System specifications

| Component | Specification |
| --- | --- |
| Laptop | ASUS TUF Gaming F16 FX607VJB |
| CPU | Intel Core 5 210H (hybrid: 4 P-cores + 4 E-cores), **E-cores enabled**; shown as "Unknown" in About This Mac |
| dGPU | NVIDIA GeForce RTX 3050 6GB Laptop GPU (NullMoth) |
| iGPU | Intel Raptor Lake-P graphics, PCI ID `8086:A7AB`, **not working** |
| RAM | 8GB DDR5 5600MHz |
| Storage | 512GB Western Digital NVMe SSD (single-disk dual boot with Windows 11) |
| Wi-Fi / Bluetooth | Realtek RTL8852BE |
| Ethernet | Realtek RTL8111 |

## What works

- **dGPU:** RTX 3050 with Metal 3 hardware acceleration via NullMoth.
- **Built-in display:** full resolution and refresh rate through dGPU routing.
- **Audio:** built-in speakers and 3.5mm headphone jack (AppleALC).
- **Wi-Fi and Bluetooth:** RTL8852BE works on standard home networks (WPA2/WPA3), Bluetooth audio and peripherals work.
- **Ethernet:** works with the kext version listed above (see "Ethernet").
- **Power and thermals:** CPU power management, battery readout, fan telemetry and an 80% charge limit through AsusBridge.
- **Input:** keyboard (including backlight and Fn hotkeys) and touchpad with multi-touch gestures.
- **USB and storage:** all USB-A / USB-C ports, NVMe SSD at full speed.
- **Webcam:** works in Safari.

## BIOS settings

| Setting | Value |
| --- | --- |
| Secure Boot | Disabled |
| Fast Boot | Disabled |
| CSM | Disabled (required by NullMoth) |
| Above 4G Decoding | Enabled (required by NullMoth) |
| Graphics mode (BIOS / Armoury Crate) | Ultimate / dGPU only |
| E-cores | Enabled |
| VMD Controller | Disabled (or AHCI mode if supported) |
| CFG Lock / VT-d | Handled through OpenCore quirks |

Please check these against your own BIOS before booting; names differ between BIOS versions.

## E-cores enabled: the EXITBS:START freeze

With E-cores enabled in BIOS, OpenCore stopped at `EXITBS:START` and macOS never started. With E-cores disabled it booted normally.

**Things that did not change anything:** `cpus=1`, disabling CpuTopologyRebuild.kext, disabling the force-HT kernel patch, disabling `APIC.aml`, switching the GPU mode between dGPU-only and Standard.

**What fixed it:** a small change in the CPUID masking.

```text
Kernel -> Emulate
  Before: Cpuid1Data = [old value]
          Cpuid1Mask = [old value]
  After:  Cpuid1Data = [new value]
          Cpuid1Mask = [new value]
```

[One or two sentences on why this value works or what it changes.]

## Ethernet (RTL8111)

The latest `RealtekRTL8111.kext` release I downloaded did not work on macOS 15.8.x ([symptom: no link / no IP / drops]). The version bundled by OpCore Simplify, **[version number]**, works. The problematic version is **[version number]**.

## Benchmarks (Geekbench 7.1.0)

All tests were run on AC power with Armoury Crate set to Ultimate (dGPU) mode.

| Test | Windows 11 | macOS Sequoia 15.8.3 (NullMoth) | macOS vs Windows |
| --- | --- | --- | --- |
| CPU, single-core | 1,854 | [verify macOS score] | [verify] |
| CPU, multi-core | 8,154 | 5,643 | 69% |
| GPU, Vulkan (Windows) / Metal (macOS) | 46,827 | 45,135 | 96% (about 3.6% lower) |
| GPU, OpenCL | 48,652 | 26,984 | 55% |

Notes: macOS does not schedule P-cores and E-cores as efficiently as Windows, which lowers CPU scores. OpenCL is deprecated on macOS and runs through a translation layer.

Screenshots (About This Mac, CPU and GPU comparisons) are in the `screenshots/` folder.

## Known limitations

- **Intel iGPU (not working):** the Raptor Lake-P iGPU (`8086:A7AB`) gets no acceleration. WhateverXe 0.8.2 with WhateverXe Graphics 0.4.4 was tested, including with the dGPU disabled and in Standard GPU mode, without success. The internal display is therefore driven by the dGPU, so the laptop must stay in Ultimate / dGPU only mode.
- **Deep sleep / hibernation:** entering deep sleep causes a kernel panic and a reboot (PCIe power-state mismatch under NullMoth). Use normal screen lock / display sleep.
- **Wi-Fi on enterprise networks:** 802.1X / Eduroam networks are not supported by the RTL8852BE driver.
- **NVIDIA HDMI / DisplayPort audio:** audio over external displays through the dGPU does not work.
- **Heavy Java mods (for example Minecraft MTR, Create):** occasional stutter, likely from P-core / E-core scheduling.
- **Browsers:** the camera works in Safari. Chromium-based browsers apply stricter sandbox / TCC limits. Chrome may flicker briefly when switching between dark and light themes.
- **UI artifacts:** because the CPU is reported as "Unknown" and the display is routed through the dGPU, About This Mac or System Settings may occasionally render blank or show dark text on light backgrounds.
- **Battery charge limit in dual boot:** the 80% limit is handled by the ASUS embedded controller, so switching between Windows and macOS can occasionally reset it.

## Using AsusBridge

On modern ASUS laptops I use AsusBridge instead of AsusSMC, for [short reason: which feature worked / what did not work with AsusSMC].

## License

The configuration files in this repository are released under the MIT License. The kexts, drivers and tools referenced below keep their own licenses.

## Acknowledgments

- [AsusBridge](https://github.com/ixboss/asus-control-macos): ASUS EC telemetry, keyboard backlight and battery charge limit.
- [Airport_RTW89](https://github.com/FranciumOS-org/Airport_RTW89): Realtek RTL8852BE Wi-Fi on macOS.
- NullMoth ([link]): Metal 3 acceleration for the RTX 3050.
- [WhateverXe](https://github.com/b00t0x/WhateverXe) by b00t0x: Intel Xe iGPU work and CpuTopologyRebuild for hybrid CPUs.
- Acidanthera: OpenCore, Lilu, VirtualSMC and other core kexts.
- Dortania: OpenCore Install Guide.
- OpCore Simplify: initial EFI generation.
