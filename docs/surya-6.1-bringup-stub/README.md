# Surya 6.1 bring-up stub

Generated from `cykside/android_kernel_xiaomi_sm6150` branch `seventeen-6.1` (Linux 6.1.177).

## Files

- `0001-surya-6.1-bringup-stub.patch` - git format-patch; apply on top of `seventeen-6.1`
- `surya-sm6150.dts` - vendor DT stub (not bootable)
- `surya.config` - vendor config fragment (stub)

## Apply

```sh
git clone https://github.com/cykside/android_kernel_xiaomi_sm6150.git
cd android_kernel_xiaomi_sm6150
git checkout seventeen-6.1
git am /path/to/0001-surya-6.1-bringup-stub.patch
```

## Status

Stub only. Still needs memory map validation, display (NT36672C), touch (NT36xxx SPI),
haptics (AW8624/DRV2624), battery auth (onewire_gpio+DS28E16), audio, camera,
modem/IPA, sensors, and MIUI compatibility work.
