# BlueRetro for the RetroScaler NGC-BlueRetro 4-port GameCube adapter

**Unofficial firmware build with Nintendo Switch 2 controller support, including
multiple NSO GameCube controllers at the same time.**

The official firmwares could not do this on the 4-port adapter: the first
Switch 2 controller connected fine, but a second one would never take port 2
(see RetroScaler/NGC-BlueRetro issues #14 and #18). This build fixes that.
Tested on a real GameCube with two Switch 2 NSO GameCube controllers on
ports 1 and 2, with working player LEDs, rumble and analog triggers.

## What is in here

Based on [darthcloud's BlueRetro](https://github.com/darthcloud/BlueRetro)
`v25.10-beta-2` (the final upstream release, commit `83e989b`), plus:

1. **RetroScaler 4-port board support**, ported from RetroScaler's own
   [NGC-BlueRetro](https://github.com/RetroScaler/NGC-BlueRetro) source and
   schematic: the four player LEDs (GPIO 14/32/33/25) follow controller
   connect/disconnect, and rumble is enabled by default on all four outputs.
2. **Six fixes cherry-picked from
   [LaserBear's BlueRetro_LBI fork](https://github.com/LaserBearIndustries/BlueRetro_LBI):**
   - Correlate SW2 SPI read responses to their request (stick calibration)
   - Clear LE LTK when a BLE link drops before HID init (stale keys locked
     controllers out permanently - the main cause of the "second controller
     never connects" problem)
   - Do not accept-list filter the passive LE scan
   - Per-device ACL reassembly, longer LE timeout, conf disconnect fix
   - Heap overflow fix in the trace buffer
   - Melee-safe combo defaults on GameCube builds (Melee's L+R+A+Start match
     reset was bit-identical to the stock power-off combo)

## Hardware

Built and tested ONLY for the **RetroScaler / Bitfunx NGC-BlueRetro 4-port
external GameCube adapter** (ESP32-WROOM, CH340 USB-C, HW1 pinout: ports on
GPIO 19/5/26/27). Do not flash it on internal adapters, HW2 devices or other
brands; single-port dongles should stick with the official builds.

## Flashing

**OTA (easiest):** adapter plugged into a powered GameCube, open
[blueretro.io](https://blueretro.io) in Chrome/Edge, connect, OTA firmware
update with `BlueRetro_hw1_gamecube.bin` from the release. After the reboot,
do a factory reset of the config in the web app so the new defaults (rumble,
Melee-safe combos) take effect, then re-pair your controllers.

**Serial (USB-C):** the combined image from the release flashes at address
`0x0` with NodeMCU-PyFlasher (RetroScaler's documented method) or esptool.
Individual parts: `0x1000` bootloader, `0x8000` partition-table,
`0xd000` ota_data_initial, `0x10000` app. Choosing "erase flash" also wipes
the old config and pairings, which replaces the factory-reset step.

Recovery is always possible over USB-C (the CH340 auto-reset circuit is on
board), worst case back to RetroScaler's stock
[v25.04 release](https://github.com/RetroScaler/NGC-BlueRetro/releases).

## Credits

- **Jacques Gagnon (darthcloud)** for six years of BlueRetro. This is his
  work; the branch only carries it the last mile for one specific adapter.
- **LaserBear Industries** for continuing the BT fixes after upstream was
  archived.
- **RetroScaler** for publishing their board source and schematic, which made
  the port possible.

Unofficial build, no warranty, use at your own risk. License: Apache-2.0,
same as upstream BlueRetro.
