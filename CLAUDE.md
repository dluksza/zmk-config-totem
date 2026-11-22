# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a ZMK firmware configuration repository for the TOTEM split keyboard - a 38-key column-staggered split keyboard that runs ZMK firmware on SEEED XIAO BLE or RP2040 controllers.

## Build Process

### Automated GitHub Actions Build
- Firmware builds automatically via GitHub Actions on push/PR to any branch
- Build configuration is defined in `build.yaml` which specifies:
  - `xiao_ble//zmk` board with `totem_left`, `totem_right` and `settings_reset` shields
- Built firmware files are available as artifacts in GitHub Actions
- No local build commands - all building happens in the cloud via ZMK's build system

### Manual Build (if needed locally)
- This repo uses West (Zephyr's meta-tool) for build management
- West manifest is in `config/west.yml` pointing to ZMK firmware repository
- Local builds would require West setup, but GitHub Actions is the primary build method

## Architecture

### File Structure
- `config/totem.keymap` - Main keymap configuration with 5 layers and home row mods
- `config/totem.conf` - ZMK feature configuration for both halves (sleep, pointing, battery reporting)
- `config/totem_left.conf` - Configuration for the left (central) half only (smooth scrolling, split battery levels)
- `config/west.yml` - West manifest pointing to ZMK main branch
- `build.yaml` - GitHub Actions build configuration
- `boards/shields/totem/` - Hardware definition files for the TOTEM keyboard (no default keymap; `config/totem.keymap` is the only keymap)

### Keymap Configuration (`config/totem.keymap`)
- **Layers**: BASE, NUM, NAV, CODE, ADJ (5 total layers)
- **Home Row Mods**: One hold-tap behavior per finger, each with its own timing:
  - Left hand, home row: A=Shift, S=Ctrl, D=Alt, F=GUI
  - Right hand: Shift is on the home row (`;`). GUI, Alt and Ctrl are on the top row (U, I, O)
  - Tapping terms: 220ms base (`TAPPING_TERM_MS`), 200ms for both pinkies, 260ms for left ring, 240ms for right middle, 280ms for right ring
  - All behaviors use the `tap-preferred` flavor, except right ring, which uses `balanced`
  - The tuning history and the reasons are in `docs/home-row-mod-tuning.md`
- **Key Definitions**: KEYS_L, KEYS_R, THUMBS for hold-trigger-key-positions
- **Custom Behaviors**: Individual home row mod behaviors per finger with specific timing

### ZMK Features Enabled
`config/totem.conf` (both halves):
- Sleep mode with 1-hour timeout
- Pointing support (mouse keys on the NAV layer)
- Battery reporting every 2 minutes

`config/totem_left.conf` (central half only):
- Smooth scrolling
- Split keyboard battery level fetching and proxying

### Hardware Definition (`boards/shields/totem/`)
- Shield definitions for left/right halves
- Device tree overlays and configuration (`kscan0` is a `wakeup-source`, so a key press wakes the keyboard from deep sleep)
- Kconfig settings for the TOTEM keyboard shield

## Development Workflow

1. **Keymap Changes**: Edit `config/totem.keymap` for layout modifications
2. **Feature Changes**: Edit `config/totem.conf` for ZMK feature configuration. Put central-only options in `config/totem_left.conf`
3. **Hardware Changes**: Modify files in `boards/shields/totem/` (rare)
4. **Testing**: Push to GitHub - firmware builds automatically
5. **Deployment**: Download firmware from GitHub Actions artifacts and flash to keyboard halves

## Key Architecture Notes

- This is a user config repository that imports ZMK as a dependency
- No local compilation setup needed - relies entirely on GitHub Actions
- Split keyboard setup requires separate firmware for left/right halves
- Home row mods are extensively customized with per-finger timing optimization
- Pointing support is enabled for mouse keys only. The shield defines no trackball or trackpad