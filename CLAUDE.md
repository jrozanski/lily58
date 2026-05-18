# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ZMK firmware configuration for a Lily58 split keyboard. Hardware: nice!nano v2 controllers, nice!view displays, EC11 rotary encoder on the left side.

## Building Firmware

Firmware is built via GitHub Actions — push to the repo and download artifacts from the Actions tab. There is no local build setup; the workflow uses the official ZMK build pipeline:

```
# Triggered automatically on push, or manually via workflow_dispatch
# Artifacts: lily58_left-nice_nano_v2-zmk.uf2, lily58_right-nice_nano_v2-zmk.uf2, settings_reset-nice_nano_v2-zmk.uf2
```

To flash: put the nice!nano into bootloader mode (double-tap reset), it mounts as a USB drive, then copy the `.uf2` file onto it.

## Repository Structure

- `config/lily58.conf` — Kconfig options (BLE, power, encoder, display settings)
- `config/lily58.keymap` — Keymap with layers and combos (devicetree syntax)
- `config/west.yml` — ZMK dependency pinned to `v0.3`
- `build.yaml` — Defines the three build targets (left, right, settings_reset)

## Split Keyboard Architecture

- **Right side = BLE central**: connects to the host (PC/phone)
- **Left side = BLE peripheral**: connects to the right side only
- Both halves use the `nice_view_adapter nice_view` shield for the display
- The left half includes the `studio-rpc-usb-uart` snippet enabling ZMK Studio over USB

## Keymap Layers

| Index | Name   | Activated by        |
|-------|--------|---------------------|
| 0     | BASE   | default             |
| 1     | NAV    | hold left thumb key |
| 2     | SYMBOL | hold right thumb key|
| 3     | MOUSE  | both thumb keys held (mo 3 on both NAV and SYMBOL) |

The EC11 encoder on the left side is bound to volume up/down on all layers via `sensor-bindings`.

## Key Config Options

`CONFIG_ZMK_SLEEP` and `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` control deep sleep on the peripheral. Deep sleep can cause the left side to fail to reconnect after waking — if the left side goes offline and requires a power cycle, investigate these settings first.

ZMK version is pinned in `config/west.yml`. To upgrade, change the `revision` field and update the workflow ref in `.github/workflows/build.yml` to match.
