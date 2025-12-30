# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a ZMK firmware configuration for the TOTEM split keyboard, a 38-key column-staggered split keyboard designed to work with SEEED XIAO BLE controllers. The repository contains keyboard layout definitions, hardware configuration files, and GitHub Actions workflows for automated firmware builds.

## Development Workflow

### Building Firmware

Firmware is built automatically via GitHub Actions on every push, pull request, or manual workflow dispatch. The build process:

1. Triggers on push/PR/manual dispatch (`.github/workflows/build.yml`)
2. Uses ZMK's official build workflow: `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`
3. Builds firmware for both keyboard halves based on `build.yaml` configuration:
   - Left half: `seeeduino_xiao_ble` board + `totem_left` shield
   - Right half: `seeeduino_xiao_ble` board + `totem_right` shield
4. Outputs `firmware.zip` in GitHub Actions artifacts containing:
   - `totem_left-seeeduino_xiao_ble-zmk.uf2`
   - `totem_right-seeeduino_xiao_ble-zmk.uf2`

**There is no local build command** - all builds happen in GitHub Actions.

### Flashing Firmware

After downloading the firmware from GitHub Actions:

1. Connect left half to PC
2. Press reset button twice to enter bootloader mode
3. Keyboard appears as mass storage device
4. Drag and drop `totem_left-seeeduino_xiao_ble-zmk.uf2` to the storage device
5. Repeat for right half with `totem_right-seeeduino_xiao_ble-zmk.uf2`

### Making Keymap Changes

The primary file to edit is `config/totem.keymap`. After committing and pushing changes, GitHub Actions automatically builds new firmware.

## Architecture

### File Structure and Responsibilities

- **`build.yaml`**: Defines which board/shield combinations to build. Specifies that firmware should be built for both `totem_left` and `totem_right` shields using the `seeeduino_xiao_ble` board.

- **`config/totem.keymap`**: Main user-editable keymap file defining all layers, combos, macros, and key bindings. This is the primary file users should modify to customize their keyboard layout.

- **`config/totem.conf`**: User-level configuration options (currently disables USB logging).

- **`config/boards/shields/totem/`**: Hardware definition files for the TOTEM keyboard shield:
  - `totem.dtsi`: Core devicetree include defining the matrix transform (key positions) and kscan configuration (GPIO matrix scanning setup)
  - `totem_left.overlay` / `totem_right.overlay`: Left/right specific hardware configurations
  - `totem_left.conf` / `totem_right.conf` / `totem.conf`: Side-specific configuration files (currently empty)
  - `Kconfig.defconfig` / `Kconfig.shield`: Kconfig files for shield configuration
  - `totem.zmk.yml`: Shield metadata (name, type, required board, siblings)
  - `totem.keymap`: Shield-level default keymap (users should edit `config/totem.keymap` instead)

### Keymap Architecture

The keymap (`config/totem.keymap`) defines 6 layers:

0. **BASE** - Main typing layer (Colemak-DH layout with home row mods)
1. **NAV** - Navigation, numbers, and brackets (accessed via layer-tap on TAB)
2. **SYM** - Symbols, special characters, media controls, and international characters
3. **ADJ** - Adjust layer for system functions (Bluetooth, reset, bootloader, F-keys)
4. **TVP1** - TVPaint layer 1 (application-specific shortcuts for TVPaint animation software)
5. **TVP2** - TVPaint layer 2 (extended TVPaint shortcuts)

**Layer Access Patterns:**
- NAV layer: Hold TAB key (`&lt NAV TAB`)
- SYM layer: Hold ESC key (`&lt SYM ESC`)
- ADJ layer: Accessed via momentary hold from NAV or SYM layer (`&mo ADJ`)
- TVP1 layer: Toggle via combo (keys 11+12+13 simultaneously)
- TVP2 layer: Hold L key while in TVP1 layer (`&lt TVP2 L`)

**Key Features:**
- Home row mods on BASE layer: A(GUI), R(ALT), S(CTRL), T(SHIFT) on left; N(SHIFT), E(CTRL), I(ALT), O(GUI) on right
- Mod-tap configuration: 170ms tapping term, 100ms quick-tap, tap-preferred flavor
- Combos: ESC (keys 0+1), TVP toggle (keys 11+12+13)
- Macros: `gif` macro types "#gif" for quick access to GIPHY search

### Matrix Transform

The keyboard uses a 4-row × 10-column matrix (defined in `totem.dtsi`). The physical layout is 38 keys total:
- 10 keys × 2 rows (top two rows) = 20 keys
- 12 keys × 1 row (third row with outer thumb keys) = 12 keys
- 6 keys (thumb cluster, 3 per side) = 6 keys

The matrix transform maps physical switch positions to logical key positions in the keymap.

## ZMK Documentation References

When modifying the keymap, refer to ZMK documentation at https://zmk.dev/docs/:
- Key codes: https://zmk.dev/docs/codes/
- Behaviors: https://zmk.dev/docs/behaviors/
- Configuration: https://zmk.dev/docs/config/

## Important Notes

- This repository is a fork-and-customize template - users fork it, modify `config/totem.keymap`, push changes, and download firmware from GitHub Actions
- There is no local build environment setup - all builds happen via GitHub Actions
- The SEEED XIAO BLE does not have a settings reset firmware file (noted in `build.yaml`)
- USB logging is disabled by default in `config/totem.conf`
