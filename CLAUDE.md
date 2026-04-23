# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZMK firmware configuration for two Corne split keyboards, both using Nice!Nano v2 controllers:

- **BOMv2 Corne** (Typeractive Corne Wireless, choc): uses the upstream ZMK `corne` shield with a Nice!View ePaper display. No RGB LEDs. Configured via `config/corne.keymap` and `config/corne.conf`.
- **Corne V3** (legacy, custom shield): uses a repo-local shield at `config/boards/shields/corne_v3/`. Has 27 WS2812 LEDs per side and an SSD1306 OLED. Configured via `config/corne_v3.keymap` and `config/corne_v3.conf`.

ZMK is pinned to **v0.3** in `config/west.yml`.

## Building the Firmware

GitHub Actions builds on every push. `build.yaml` defines 4 build targets:

- `corne_v3_left` / `corne_v3_right` — legacy V3 firmware
- `corne_left nice_view_adapter nice_view` (+ `studio-rpc-usb-uart` snippet on the left/central half) — BOMv2 Corne
- `corne_right nice_view_adapter nice_view` — BOMv2 Corne right half

Firmware is emitted as GitHub Actions artifacts. Flash by double-tapping reset on each half and copying the `.uf2` to the mounted drive.

## Repository Structure

```
config/
├── boards/shields/corne_v3/     # Custom V3 shield (RGB + SSD1306 OLED)
│   ├── corne_v3.dtsi            # Matrix, OLED, kscan
│   ├── corne_v3_left.overlay    # Left half GPIOs
│   ├── corne_v3_right.overlay   # Right half GPIOs
│   ├── boards/nice_nano_v2.overlay  # WS2812 LED strip (27 LEDs)
│   ├── Kconfig.defconfig        # I2C, display, split
│   └── corne_v3.zmk.yml         # Shield metadata
├── corne_v3.keymap              # V3 keymap (6 layers, includes RGB layer)
├── corne_v3.conf                # V3 config
├── corne.keymap                 # BOMv2 Corne keymap (5 layers, no RGB)
├── corne.conf                   # BOMv2 Corne config (name, studio, TX power)
└── west.yml                     # West manifest (ZMK pinned to v0.3)
zephyr/module.yml                # Zephyr module definition (sets board_root for V3 shield)
```

The BOMv2 Corne has **no custom shield directory**. It relies entirely on the upstream ZMK `corne`, `nice_view_adapter`, and `nice_view` shields. This matches Typeractive's own config pattern.

## Keymaps

### BOMv2 Corne (`config/corne.keymap`)

5 layers:

1. **ALPHA** (0): QWERTY with homerow mods (LCTRL/LALT/LGUI/LSHFT on ASDF, mirrored on JKL;)
2. **NUM** (1): numbers + arrows
3. **SYMBOL** (2): punctuation
4. **FUNC** (3): F1–F10
5. **BT** (4): `&bt BT_SEL N`, `&bt BT_CLR`, `&out`

Layer access is chained via `&mo`: NUM is the left thumb layer-tap, SYMBOL is the right, FUNC is reached from SYMBOL, BT is reached from FUNC.

Custom behaviors: `hm` (homerow mods, 175ms tapping-term, tap-preferred), `lt_spc`, `lt_ret`, `esc_hold`, `hyper` macro (LSHFT+LCTRL+LALT+LGUI). A caps-word combo triggers on key positions 16+19.

### Corne V3 (`config/corne_v3.keymap`)

Same 5 base layers plus layer 5 (**RGB**) with `&rgb_ug` bindings for underglow control.

## Hardware Notes

| | BOMv2 Corne | Corne V3 |
|---|---|---|
| Controller | Nice!Nano v2 | Nice!Nano v2 |
| Display | Nice!View (SPI ePaper, 160×68) | SSD1306 (I2C OLED, 128×32) |
| RGB | none | 27 WS2812 LEDs per side (P0.06 SPI, GRB) |
| Central half | left | left |
| Switches | choc | MX |
| ZMK Studio | enabled | not enabled |

Pinout for the BOMv2 Corne matches the upstream ZMK `corne` shield (rows `pro_micro 4–7`, columns `pro_micro 14–21`). That's why no custom shield is needed.

## Making Changes

### Editing a keymap

- BOMv2 Corne: edit `config/corne.keymap`
- Corne V3: edit `config/corne_v3.keymap`

Each layer has 3 rows of 12 keys followed by 3 thumb keys per side (left then right). Use ZMK behavior bindings: `&kp`, `&mo`, `&bt`, `&out`, `&rgb_ug` (V3 only), `&caps_word`, plus the custom behaviors defined in the same file.

### Adjusting RGB LED count (V3 only)

Edit `config/boards/shields/corne_v3/boards/nice_nano_v2.overlay` and change `chain-length = <27>;`.

### Configuration options

- BOMv2 Corne: `config/corne.conf` (keyboard name, TX power, debounce, ZMK Studio)
- Corne V3: `config/corne_v3.conf` (display, RGB, power management, keyboard name)

### Updating ZMK

Edit `revision:` in `config/west.yml`. Currently pinned to `v0.3`.

## Testing Changes

1. Push → GitHub Actions runs all four builds
2. Download `.uf2` artifacts from the Actions run
3. Double-tap reset on each half and copy the matching `.uf2` to the mounted drive
