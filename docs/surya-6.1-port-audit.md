# Surya (POCO X3 NFC / J20C) 4.14 -> 6.1 Port Audit

## Status

This is an **audit/roadmap**, not a completed port. A full 4.14.357-openela (sm6150/sdmmagpie) -> 6.1.x sm6150 bring-up for surya is too large to complete safely in a single session.

## Current base (this repo)

- Repo: `cykside/msm-4.14`
- Branch: `arena/6ccacdbd-msm-4-14`
- Kernel: `4.14.357-openela` (`Makefile`)
- Platform: Qualcomm sm6150 / sdmmagpie
- Device: Xiaomi POCO X3 NFC (`surya`, also related to `karna`/J20 family)

## Target base (user-provided)

- Repo: `xiaomi-sm6150/android_kernel_xiaomi_sm6150`
- Branch: `seventeen-6.1`
- Kernel: `6.1.177` (`Makefile` verified)
- Surya support in target: **not found** by quick scan (`surya`/`karna` files/defconfig absent in tree).

## Surya device identity in this 4.14 tree

Surya is not represented by a `*surya*.dts`; it is carried through the sdmmagpie IDP MIUI overlay:

- `arch/arm64/boot/dts/qcom/sdmmagpie-idp-overlay-miui.dts`
  - `model = "IDP"`
  - `compatible = "qcom,sdmmagpie-idp", "qcom,sdmmagpie", "qcom,idp"`
  - `qcom,msm-id = <365 0x0>`
  - `qcom,board-id = <34 0>`
  - Includes: `sdmmagpie-idp.dtsi`, `sdmmagpie-audio-overlay.dtsi`, `haptic.dtsi`, `touchscreen.dtsi`
  - Xiaomi-specific nodes: `xiaomi,onewire_gpio`, `maxim,ds28e16`
  - Displays active: `dsi_nt36672c_huaxing_fhd_video_display`, `dsi_nt36672c_tianma_fhd_video_display`

Build wiring:
- `arch/arm64/boot/dts/qcom/Makefile`: builds `sdmmagpie-idp-overlay-miui.dtbo` with base `sdmmagpie.dtb`.

## Device-specific files/configs to port

### Defconfigs

- `arch/arm64/configs/surya_defconfig`
- `arch/arm64/configs/vendor/surya-stock_defconfig`

Notes:
- Review all CONFIG differences vs target 6.1 sm6150 defconfig; do not copy 4.14 config symbols blindly.
- `kernel/Makefile` in this tree spoofs `/proc/config.gz` to `vendor/surya-stock_defconfig`; check whether target has similar behavior/expectations.

### DTS / DTBO

- `arch/arm64/boot/dts/qcom/sdmmagpie-idp-overlay-miui.dts`
- `arch/arm64/boot/dts/qcom/haptic.dtsi`
- `arch/arm64/boot/dts/qcom/touchscreen.dtsi`
- Base includes to audit/adapt: `arch/arm64/boot/dts/qcom/sdmmagpie-idp.dtsi`, `sdmmagpie-idp-overlay.dts`, sdmmagpie audio/display/pinctrl/gpu/camera includes as needed.

The touchscreen DTS is large (857 lines) and contains Novatek NT36672c MP criteria tables for "Xiaomi J20 672C TM6.67\" module".

### Device-specific drivers present in 4.14

| Component | Compatible / purpose | Paths in this tree |
|---|---|---|
| Touchscreen | `novatek,NVT-ts-spi` (NT36xxx SPI) | `drivers/input/touchscreen/nt36xxx_spi/` (`nt36xxx.c`, `nt36xxx.h`, `nt36xxx_ext_proc.c`, `nt36xxx_fw_update.c`, `nt36xxx_mem_map.h`) |
| Haptic | `awinic,aw8624_haptic` | `drivers/misc/aw8624_haptic/` (`aw8624.c`, `aw8624.h`, `aw8624_reg.h`) |
| Haptic | `ti,drv2624` (DRv2624 support in same haptic DTS node) | `drivers/misc/drv2624_haptic/` (`drv2624.c`, `drv2624.h`, `drv2624_parser_interfaces.h`) |
| Battery auth / onewire | `xiaomi,onewire_gpio` | `drivers/power/supply/maxim/onewire_gpio.c`, `onewire_gpio.h` |
| Battery auth | `maxim,ds28e16` | `drivers/power/supply/maxim/ds28e16.c`, `ds28e16.h` |

These drivers are 4.14 downstream/QCOM-oriented; they will likely need API adaptation for 6.1 (input, power supply, spi, gpiod, regulator, pmic, DRM/notifier, sysfs/proc interfaces).

## Areas requiring manual porting/review

1. **Boot/DT bring-up**: sm6150/sdmmagpie -> 6.1 DTS style, board-id/msm-id matching, bootloader dtbo expectations.
2. **Display**: NT36672C Huaxing/Tianma panels; 6.1 uses different DRM/MSM panel/DSI bindings than 4.14.
3. **Touch**: NT36xxx SPI driver port or replacement with upstream/6.1-compatible driver; firmware/MP config handling.
4. **Haptics**: AW8624/DRV2624 driver port; check if mainline/target has equivalent drivers.
5. **Power/battery/auth**: onewire_gpio + DS28E16; often MIUI-specific and fragile across kernel versions.
6. **Audio**: sdmmagpie audio overlay, machine driver, codec/amp routing for surya.
7. **Camera**: sensor/power/CCI nodes and any surya-specific camera hacks.
8. **Connectivity/modem/RIL**: IPA/RMNET/modem dependencies; usually hardest for major jumps.
9. **Thermal/power/perf**: schedutil/schedtune/WALT/cpufreq differences; 6.1 scheduler/energy model is different.
10. **SELinux/vendor userspace**: MIUI 14 Android 12 userspace expects specific kernel interfaces; 6.1 compatibility may require extra patches.

## Recommended incremental plan

1. **Inventory**: diff surya/sdmmagpie-idp-miui device files against target sm6150 6.1 device support.
2. **Minimal boot DTS**: add surya board-id/compatible/dtbo in 6.1 style; boot to UART/dmesg.
3. **Core peripherals**: USB/ADB, storage/UFS, PMIC, regulators, simplefb/display.
4. **Input/haptics**: touch then haptics.
5. **Network/modem**: Wi-Fi, then modem/IPA/RIL.
6. **Audio/camera/battery auth**: last, highest regression risk.
7. **MIUI compatibility layer**: only as needed, keep minimal.

## What is feasible in this session

- This audit document.
- Extracting a more detailed file/commit inventory from this 4.14 tree.
- Preparing initial surya DTS/defconfig stubs *if* a 6.1 target tree is present in workspace (not done here).
- Backporting/tuning on 4.14 (already in progress).

## What is not feasible in this session

- Complete, bootable, MIUI-compatible surya port to Linux 6.1.
- Full QCOM downstream subsystem migration 4.14 -> 6.1.
- Modem/camera/audio bring-up validation without device-side iterative testing.
