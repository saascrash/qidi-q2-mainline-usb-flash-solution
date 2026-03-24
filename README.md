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
3. On the Q2 touchscreen, go to **Settings** → **Offline Update** to start the update

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

## Background: Why This Exists

The QIDI Q2 ships with a heavily modified Klipper fork. Getting mainline Klipper running on it requires dealing with several hardware and software quirks that aren't documented anywhere. This package is the result of reverse-engineering the stock firmware and finding a path that doesn't require specialized hardware.

### The Problem

The Q2's main MCU is a **GD32F425** — a Chinese clone of the STM32F407. It's mostly compatible, but its USB OTG peripheral has a silicon bug: it needs extra initialization delays that real STM32s don't. Without a workaround, the MCU fails to enumerate over USB and Klipper can't communicate with it.

Additionally, Klipper doesn't natively support the **CS1237** load cell ADC used on the Q2's bed probe. Stock QIDI firmware handles this via proprietary `.so` extensions that aren't available for mainline Klipper.

### The Hard Way (ST-Link + Katapult)

The "proper" approach to running custom firmware on STM32/GD32 boards is:

1. Buy an ST-Link programmer (~$10)
2. Open the printer and connect the ST-Link to the MCU's SWD header
3. Flash the Katapult bootloader (replaces the stock 32KiB bootloader with an 8KiB one)
4. Flash Klipper via Katapult over USB (repeatable, no ST-Link needed after first flash)
5. Repeat for the THR board (but via UART instead of USB)

This works, but it requires hardware, disassembly, and comfort with SWD debugging. It also changes the bootloader, which means the stock USB offline update mechanism no longer works. If something goes wrong, you need the ST-Link again to recover.

### The Easy Way (This Package)

We discovered that the stock Q2 update mechanism is simpler than expected:

- **SOC updates** are just `.deb` packages — standard Debian packages installed via `dpkg`. No encrypted containers, no signatures.
- **MCU/THR updates** use QIDI's `hid-flash` tool, which speaks USB HID to the stock 32KiB bootloader. The stock bootloader is perfectly capable of flashing custom Klipper — it doesn't care what binary you give it.
- **The USB offline update flow** (`QD_Update/` folder on a USB stick) triggers all three: SOC .deb install, MCU flash, THR flash. All from the touchscreen, no SSH or hardware required.

### Key Decisions

**Keep the stock 32KiB bootloader** — By compiling Klipper with `FLASH_APPLICATION_ADDRESS=0x8008000` (matching the stock bootloader offset) instead of Katapult's 8KiB offset, we can flash via the existing bootloader. No ST-Link, no Katapult, no disassembly. Recovery is always possible via another USB offline update.

**Patch Klipper, don't fork it** — Our changes are minimal: a GD32 USB OTG init delay in `usbotg.c`, a Kconfig option to enable it, and the CS1237 load cell driver. Everything else is upstream mainline Klipper. This means future Klipper updates can be rebased with minimal effort.

**Pull the live rootfs instead of building from scratch** — Rather than trying to install Klipper into a rootfs image from inside a Docker container (which would require ARM64 emulation), we set up a working Q2 with mainline Klipper via KIAUH, then pulled the entire rootfs via `e2image` over SSH. This captures a known-good, tested configuration.

**Use the stock .deb format** — QIDI's update client expects a specific package naming convention (`QD_Q2_SOC_*`, `QD_Q2_MCU`, `QD_Q2_THR`). By packaging our update the same way, we piggyback on the existing update infrastructure instead of building our own.

**FTDI latency fix via udev rule** — The THR board connects via an FTDI FT232R USB-to-serial converter. The default Linux driver sets a 16ms latency timer, which causes Klipper timing errors (trsync). A udev rule (`99-ftdi-latency.rules`) sets it to 1ms. This is included in the SOC .deb and applied automatically.

## Recovery

If something goes wrong after flashing:

- **Simple recovery:** Download the official QIDI Q2 firmware from [qidi3d.com](https://qidi3d.com), place the stock `QD_Update/` folder on a USB stick, and perform another offline update to restore factory firmware.
- **Advanced recovery:** If the Q2 is unresponsive to USB updates, use an ST-Link programmer to reflash the official MCU and THR firmware directly, then perform a stock USB offline update to restore the SOC.

## Disclaimer

**USE AT YOUR OWN RISK.** This is unofficial, community-produced firmware. It is not affiliated with, endorsed by, or supported by QIDI Technology.

THIS SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES, OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT, OR OTHERWISE, ARISING FROM, OUT OF, OR IN CONNECTION WITH THIS SOFTWARE OR THE USE OR OTHER DEALINGS IN THIS SOFTWARE.

Flashing custom firmware carries inherent risk. You are solely responsible for any damage to your hardware, loss of data, or voided warranty resulting from the use of this package.

## License

This work is licensed under the [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/).

You are free to:
- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material

Under the following terms:
- **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made.
- **NonCommercial** — You may not use the material for commercial purposes.
- **ShareAlike** — If you remix, transform, or build upon the material, you must distribute your contributions under the same license.

No additional restrictions — You may not apply legal terms or technological measures that legally restrict others from doing anything the license permits.
