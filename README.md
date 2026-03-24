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
