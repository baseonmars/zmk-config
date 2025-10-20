# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ZMK (Zephyr Mechanical Keyboard) firmware configuration for a Corne V3 split keyboard using Nice Nano v2 controllers. The configuration includes:
- Custom RGB LED configuration (27 LEDs per side)
- OLED display support with WPM tracking
- Split keyboard architecture
- Bluetooth connectivity with output toggling

## Building the Firmware

This repository uses GitHub Actions for automated builds. Push to the repository triggers builds for both left and right keyboard halves.

### Build Configuration

The `build.yaml` file defines the build matrix:
- Board: nice_nano_v2
- Shields: corne_v3_left, corne_v3_right

Firmware files are generated as GitHub Actions artifacts.

## Repository Structure

```
config/
├── boards/shields/corne_v3/     # Custom Corne V3 shield definition
│   ├── corne_v3.dtsi            # Base device tree (matrix, OLED, kscan)
│   ├── corne_v3_left.overlay    # Left half GPIO configuration
│   ├── corne_v3_right.overlay   # Right half GPIO configuration
│   ├── boards/
│   │   └── nice_nano_v2.overlay # RGB LED strip configuration (27 LEDs)
│   ├── Kconfig.defconfig        # Default config (I2C, display, split)
│   └── corne_v3.zmk.yml         # Shield metadata
├── corne_v3.keymap              # Main keymap definition
├── corne_v3.conf                # Keyboard configuration
└── west.yml                     # West manifest (ZMK dependency)
zephyr/module.yml                # Zephyr module definition (sets board_root)
```

## Keymap Architecture

The keymap (`config/corne_v3.keymap`) uses a 6-layer structure:

1. **ALPHA** (layer 0): Default QWERTY layout
2. **NUM** (layer 1): Numbers and navigation arrows
3. **SYMBOL** (layer 2): Symbols and punctuation
4. **FUNC** (layer 3): Function keys (F1-F10)
5. **BT** (layer 4): Bluetooth management and output selection
6. **RGB** (layer 5): RGB underglow controls

Layer access uses momentary layer switches (`&mo N`). The BT and RGB layers are accessed via layer 3 and layer 4 respectively.

## Hardware Configuration Details

### RGB Underglow

Configured in `config/boards/shields/corne_v3/boards/nice_nano_v2.overlay`:
- 27 LEDs per side (chain-length = 27)
- SPI-based WS2812 control on pin P0.06
- Color mapping: GRB
- Controlled via layer 5 (RGB layer)

### Split Configuration

- Left side is the central (primary) controller
- Set in `Kconfig.defconfig` with `ZMK_SPLIT_ROLE_CENTRAL=y` for left shield
- Communication via Bluetooth

### Display

- OLED: SSD1306 128x32
- I2C address: 0x3c
- Features: WPM tracking enabled

### Power Settings

- External power control enabled (`ZMK_EXT_POWER=y`)
- Sleep disabled (`ZMK_SLEEP=n`)
- Idle timeout: 300 seconds
- RGB underglow disabled on start, no auto-off on USB

## Making Changes

### Modifying the Keymap

Edit `config/corne_v3.keymap`:
- Each layer has a `bindings` matrix with 3 rows of 12 keys plus 3 thumb keys per side
- Use ZMK behavior bindings like `&kp`, `&mo`, `&bt`, `&rgb_ug`, `&out`
- Bindings are defined in left-to-right, top-to-bottom order

### Adjusting RGB LED Count

If you have a different number of LEDs:
1. Edit `config/boards/shields/corne_v3/boards/nice_nano_v2.overlay`
2. Change `chain-length = <27>;` to your LED count

### Configuration Options

Edit `config/corne_v3.conf` for settings like:
- Display enable/disable
- RGB underglow behavior
- Power management
- Keyboard name

## Testing Changes

1. Push changes to trigger GitHub Actions build
2. Download firmware artifacts from Actions tab
3. Flash `.uf2` files to each keyboard half by copying to the mounted drive
4. Reset keyboard to bootloader mode (double-tap reset button)
