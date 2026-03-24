# QIDI Q2 Mainline Klipper USB Flash Solution

Drop-in USB update package for the QIDI Q2 3D printer running mainline Klipper with GD32 USB workaround and CS1237 load cell support.

## What's Inside

| File | Description |
|------|-------------|
| `QD_Update/QD_Q2_SOC_*` | SOC update (.deb) — patched Klipper klippy, Moonraker, Fluidd, configs, tools |
| `QD_Update/QD_Q2_MCU` | Main MCU firmware (GD32F425, 32KiB bootloader offset) |
| `QD_Update/QD_Q2_THR` | THR firmware (STM32F103xE, 32KiB bootloader offset) |

## Klipper Version

- **Base commit:** `88a71c3c` (upstream `Klipper3d/klipper` master)
- **Patches applied:**
  - GD32F425 USB OTG initialization workaround (fixes USB enumeration on GD32 clones)
  - CS1237 load cell sensor support

## Usage

1. Copy the `QD_Update/` folder to a USB stick (FAT32)
2. Insert USB stick into the Q2
3. The stock update client will detect and install the update automatically

## Hardware

- **SoC:** Rockchip RK3308
- **Main MCU:** GD32F425 (STM32F407 compatible)
- **THR MCU:** STM32F103xE
- **Bootloader:** Stock 32KiB (0x8008000) — no Katapult/ST-Link required

## Notes

- This package uses the stock USB update mechanism — same as official QIDI firmware updates
- The SOC .deb replaces Klipper klippy modules, Moonraker, Fluidd, and printer configs
- MCU and THR binaries are flashed automatically by the Q2's update client
- Back up your `printer.cfg` and `saved_variables.cfg` before updating
