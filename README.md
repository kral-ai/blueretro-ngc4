# BlueRetro for the RetroScaler NGC-BlueRetro 4-port GameCube adapter

**Unofficial firmware with Nintendo Switch 2 controller support — including
multiple NSO GameCube controllers at the same time.**

<p align="center"><img src="https://raw.githubusercontent.com/RetroScaler/NGC-BlueRetro/master/image/1.png" width="700" alt="RetroScaler NGC-BlueRetro 4-port adapter with 1 to 4 controllers"/></p>
<p align="center"><i>Product images from the official <a href="https://github.com/RetroScaler/NGC-BlueRetro">RetroScaler/NGC-BlueRetro</a> repo.</i></p>

## Why this exists

The Switch 2 controllers (NSO GameCube, Pro Controller 2, Joy-Con 2) use a new
BLE protocol. On the 4-port RetroScaler adapter, no available firmware could
handle more than one of them: the first NSO GameCube controller connected fine,
but a second one never took port 2 (see RetroScaler/NGC-BlueRetro issues
[#14](https://github.com/RetroScaler/NGC-BlueRetro/issues/14) and
[#18](https://github.com/RetroScaler/NGC-BlueRetro/issues/18)).

This build combines the three public pieces that together fix it:

- [darthcloud's BlueRetro](https://github.com/darthcloud/BlueRetro)
  `v25.10-beta-2` — the final upstream release with Switch 2 support
- RetroScaler's 4-port board support, ported from their published
  [source and schematic](https://github.com/RetroScaler/NGC-BlueRetro)
  (player LEDs on GPIO 14/32/33/25, rumble enabled by default)
- Six Bluetooth fixes cherry-picked from
  [LaserBear's BlueRetro_LBI fork](https://github.com/LaserBearIndustries/BlueRetro_LBI),
  among them the three that solve the multi-controller problem: clearing stale
  LE keys when a link drops early, not filtering the passive LE scan, and
  per-device ACL reassembly

## What works

Tested on a real GameCube with this exact build:

| Feature | Status |
|---|---|
| Two Switch 2 NSO GameCube controllers simultaneously | ✅ tested (firmware supports up to 4) |
| Analog sticks with factory calibration, analog triggers | ✅ tested |
| Rumble on all 4 ports (enabled by default) | ✅ tested |
| Player LEDs on the adapter follow connect/disconnect | ✅ tested |
| Player number LEDs on the controllers themselves | ✅ tested |
| Melee-safe combos | ✅ (see below) |

Inherited from upstream BlueRetro and expected to work (not all re-tested
here): Switch 2 Pro Controller, Switch 1 Pro/Joy-Con, PS3/PS4/PS5, Xbox One
S/Series X|S, Wii/Wii U Pro, 8BitDo, Retro-Bit Wireless, and the rest of the
[BlueRetro compatibility list](https://github.com/darthcloud/BlueRetro/wiki).

**Melee players:** stock BlueRetro powers the adapter off on L+R+A+Start,
which is bit-identical to Melee's match reset. This build moves the adapter
combos onto the Capture button of the NSO pad, so match resets no longer kill
your session. (Side effect: controllers without a Capture-type button, e.g.
Wii U Pro/PS3, cannot trigger adapter combos until remapped in the web config.)

**Known quirks**

- New pairings are only accepted while no controller is connected, plus about
  one minute after the first connect. Already-paired controllers reconnect
  any time with a button press. If a brand-new controller will not pair, pair
  it alone first, or hold the adapter's **BOOT** button for 3 seconds to
  re-open the pairing window.
- A Switch 2 console that is on nearby may steal the controllers back.
  Re-syncing them to the console later works as usual (fastest via USB-C).

## The hardware

<p align="center"><img src="https://raw.githubusercontent.com/RetroScaler/NGC-BlueRetro/master/image/2.png" width="450" alt="Adapter front: EN button, POWER LED, SYNC LED, BOOT button"/></p>

Only for the **RetroScaler / Bitfunx NGC-BlueRetro 4-port external GameCube
adapter** (ESP32-WROOM, CH340 USB-C, HW1 pinout: controller ports on GPIO
19/5/26/27). Do not flash internal adapters, HW2 devices or other brands;
single-port dongles should stick with the official builds.

Front edge: **EN** = reset button, **POWER** = power LED, **SYNC** = status
LED, **BOOT** = pairing window / flash-mode button.

## Flashing

Grab the files from the
[latest release](https://github.com/kral-ai/blueretro-ngc4/releases).

### A. Over the air (easiest, no cable)

1. Plug the adapter into a powered-on GameCube.
2. Open [blueretro.io](https://blueretro.io) in Chrome or Edge on a PC or
   phone with Bluetooth, click *Connect BlueRetro*.
3. Go to the OTA firmware update section, select
   **`BlueRetro_hw1_gamecube.bin`**, wait until it reaches 100 %.
   The adapter reboots itself.
4. Reconnect and check the version string: it must read
   **`v25.10-beta-11-g45e0a0a`**.
5. Do a **factory reset of the config** in the web app once, so the new
   defaults (rumble on, Melee-safe combos) take effect. Then re-pair your
   controllers.

If the update never applies (old version still shown after reboot), the
adapter silently rolled back — use the USB route below.

### B. Over USB-C (serial)

The board has a CH340 USB-serial chip with auto-reset, so no button
gymnastics are needed. Choosing **erase flash** wipes the old config and
pairings too, which replaces the factory-reset step.

**NodeMCU-PyFlasher** (the tool from RetroScaler's own manual): select the
CH340 COM port, choose
**`BlueRetro_v25.10-beta-11_4port_combined_flash_at_0x0.bin`**, baud 460800,
Dual I/O (DIO), erase flash *yes*, then *Flash NodeMCU*.

**esptool / [esptool-js](https://espressif.github.io/esptool-js/)** with the
parts zip:

| Address | File |
|---|---|
| `0x1000` | `bootloader.bin` |
| `0x8000` | `partition-table.bin` |
| `0xd000` | `ota_data_initial.bin` |
| `0x10000` | `BlueRetro_hw1_gamecube.bin` |

### Recovery

Flashing can always be redone over USB-C, even if the firmware does not boot.
Worst case, RetroScaler's stock
[v25.04 release](https://github.com/RetroScaler/NGC-BlueRetro/releases)
brings the adapter back to factory state.

## Pairing quick guide

1. First time per controller: hold the controller's sync button until the
   LEDs sweep; the adapter picks it up (pairing window: see quirks above).
2. From then on: just press a button, it reconnects by itself.
3. Connection order = player number. First to connect is player 1.

## Building from source

The repo builds with darthcloud's official container. The GitHub Actions
workflow does exactly this on every push:

```
cp configs/hw1/gamecube sdkconfig
BR_HW=_hw1 BR_SYS=_gamecube idf.py reconfigure build
```

Container image: `ghcr.io/darthcloud/idf-blueretro:v5.5.0_2024-12-02`.
The board support is gated behind `CONFIG_RETROSCALER_BLUERETRO_4LEDS_HW`,
enabled only in the `hw1/gamecube` config.

## Credits

- **Jacques Gagnon (darthcloud)** — six years of BlueRetro. This is his work;
  this repo only carries it the last mile for one specific adapter.
- **LaserBear Industries** — for continuing the Bluetooth fixes after
  upstream was archived.
- **RetroScaler** — for publishing their board source, schematic and product
  images, which made this port possible.

Unofficial build, no warranty, use at your own risk.
License: Apache-2.0, same as upstream BlueRetro.
