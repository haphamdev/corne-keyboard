# Corne Keyboard ZMK Firmware Configuration

## Project Overview

This repository contains a custom ZMK (Zephyr Mechanical Keyboard) firmware configuration for a Corne keyboard (crkbd). The configuration is optimized for the Nice!Nano v2 controller and includes advanced features like home row modifiers, multiple layers, mouse simulation, and ZMK Studio support.

## Hardware

- **Keyboard**: Corne (Crkbd) 3x6 split keyboard
- **Controller**: Nice!Nano v2 (nRF52840-based)
- **Features**:
  - Encoders support (EC11)
  - OLED display
  - Bluetooth connectivity
  - Optional RGB underglow (currently disabled)
  - Mouse/pointing device simulation

## Build System

### GitHub Actions

The project uses GitHub Actions for automated firmware builds:
- Located in `.github/workflows/build.yml`
- Triggers on: push, pull_request, workflow_dispatch
- Uses ZMK's official build workflow (v0.3)
- Build configuration: `build.yaml`

### Build Configuration

The `build.yaml` file specifies:
- **Board**: nice_nano_v2
- **Shields**: corne_left, corne_right
- **Special Features**:
  - ZMK Studio enabled (`-DCONFIG_ZMK_STUDIO=y`)
  - Studio RPC over USB UART (`studio-rpc-usb-uart` snippet)
- **Artifacts**:
  - `corne_left_studio`
  - `corne_right_studio`

## Firmware Features

### 1. Key Debouncing (`config/corne.conf:2-3`)

```
CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=3
CONFIG_ZMK_KSCAN_DEBOUNCE_RELEASE_MS=3
```

Fast debounce timing (3ms) for responsive key presses.

### 2. Bluetooth Configuration (`config/corne.conf:9-13`)

- **TX Power**: +8dBm for extended range
- **Windows Compatibility**: Battery reporting fix
- **Connection Stability**: Enhanced buffer sizes (512 bytes ACL RX, 251 max data length)
- **Bond Management**: Preserves paired devices across reboots

### 3. Display Configuration (`config/corne.conf:15-17`)

- OLED display enabled
- Dedicated work queue for display updates

### 4. Power Management (`config/corne.conf:19-26`)

- **Idle Timeout**: 60 seconds (60000ms)
- **Sleep Mode**: Enabled
- **Sleep Timeout**: 5 minutes (300000ms)
- External power control and USB support

### 5. ZMK Studio (`config/corne.conf:28-31`)

ZMK Studio is enabled for live keymap editing:
- No locking required
- Stays unlocked on disconnect
- Access via web interface

### 6. Mouse/Pointing Device (`config/corne.conf:54`)

Pointing device support enabled for mouse emulation features.

### 7. RGB Underglow (Currently Disabled)

Configuration exists but is commented out (`config/corne.conf:33-41`):
- WS2812 LED strips
- Max brightness: 30%
- Auto-off when idle

## Keymap Layout

The keymap is defined in `config/corne.keymap` with 7 layers:

### Layer 0: Base Layer (QWERTY)

```
┌──────┬─────┬─────┬─────┬─────┬─────┐   ┌─────┬─────┬─────┬─────┬─────┬──────┐
│      │  Q  │  W  │  E  │  R  │  T  │   │  Y  │  U  │  I  │  O  │  P  │      │
├──────┼─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┼──────┤
│      │ A/⌘ │ S/⌥ │ D/⌃ │ F/⇧ │  G  │   │  H  │ J/⇧ │ K/⌃ │ L/⌥ │ ;/⌘ │      │
├──────┼─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┼──────┤
│      │  Z  │  X  │  C  │  V  │  B  │   │  N  │  M  │  ,  │  .  │  /  │      │
└──────┴─────┴─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┴─────┴──────┘
                    │ESC/4│ `/3 │ SPC │   │RET/5│BSPC/1│DEL/2│
                    └─────┴─────┴─────┘   └─────┴─────┴─────┘
```

Features **home row modifiers** with optimized settings:
- Left hand: GUI, ALT, CTRL, SHIFT (A, S, D, F)
- Right hand: SHIFT, CTRL, ALT, GUI (J, K, L, ;)
- Balanced flavor with 280ms tapping term
- 150ms prior idle requirement
- 175ms quick tap threshold

### Layer 1: Numbers (`config/corne.keymap:58-73`)

Numpad layout with:
- Left side: Numpad 0-9 with operators (+, -, *, /)
- Right side: Bluetooth selection (BT0-BT2) and modifiers

### Layer 2: Symbols (`config/corne.keymap:75-90`)

Symbol layer featuring:
- Parentheses, braces, brackets
- Special characters (!@%&-)
- Caps lock on right side

### Layer 3: Navigation (`config/corne.keymap:92-101`)

Navigation controls:
- Arrow keys (vim-style on right hand)
- Home, End, Page Up, Page Down
- Left hand retains modifiers

### Layer 4: Mouse (`config/corne.keymap:103-112`)

Mouse emulation layer:
- Mouse movement (HJKL style)
- Scroll controls
- Left/right/middle click on thumb keys
- Modifiers available on left hand

### Layer 5: Function (`config/corne.keymap:114-123`)

Function keys and media controls:
- F1-F12 keys
- Media playback (play/pause, volume)
- Brightness controls

### Layer 6: Bluetooth (`config/corne.keymap:125-134`)

Bluetooth management layer (activated via conditional layers):
- Device selection (BT0-BT2)
- Clear bonds
- Activated when layers 2 and 4 are active simultaneously

## Custom Behaviors

### Home Row Modifiers (`config/corne.keymap:14-36`)

Two custom behaviors defined:

**hml (home_row_mod_left)**:
- For left hand home row
- Hold-trigger positions: Right side keys + thumb cluster
- Prevents accidental activation when typing on same hand

**hmr (home_row_mod_right)**:
- For right hand home row
- Hold-trigger positions: Left side keys + thumb cluster
- Mirror configuration of hml

Both use:
- Balanced flavor for consistent feel
- Hold-trigger-on-release for better modifier behavior
- Optimized timing values for typing comfort

## File Structure

```
corne-keyboard/
├── .github/
│   └── workflows/
│       └── build.yml          # GitHub Actions build workflow
├── boards/                    # Custom board definitions (if any)
├── config/
│   ├── corne.conf            # Main configuration file
│   ├── corne.keymap          # Keymap definition
│   ├── corne.dtsi            # Device tree include
│   ├── corne.overlay         # Hardware overlay
│   └── west.yml              # West manifest for ZMK
├── zephyr/                   # Zephyr RTOS files
└── build.yaml                # Build matrix configuration
```

## Development Workflow

### Building Locally

To build this firmware locally, you need the ZMK development environment:

1. Install West and dependencies
2. Initialize workspace: `west init -l config/`
3. Update modules: `west update`
4. Build left half: `west build -d build/left -b nice_nano_v2 -- -DSHIELD=corne_left -DCONFIG_ZMK_STUDIO=y`
5. Build right half: `west build -d build/right -b nice_nano_v2 -- -DSHIELD=corne_right -DCONFIG_ZMK_STUDIO=y`

### Building via GitHub Actions

Simply push changes to trigger an automatic build. Firmware files will be available as workflow artifacts.

### Flashing Firmware

1. Download the `.uf2` files from GitHub Actions artifacts
2. Connect keyboard half via USB while holding RESET
3. Copy the appropriate `.uf2` file to the mounted drive
4. Repeat for the other half

## ZMK Studio Usage

With ZMK Studio enabled, you can modify the keymap in real-time:

1. Connect keyboard via USB
2. Visit the ZMK Studio web interface
3. Make changes to keymap visually
4. Changes take effect immediately (but are not persisted to this repo)

## Configuration Tips

### Adjusting Home Row Modifiers

If home row mods feel too sensitive or not sensitive enough, adjust these values in `config/corne.keymap`:

- `tapping-term-ms`: Increase for easier holds, decrease for faster taps
- `require-prior-idle-ms`: Increase to prevent accidental mod activation
- `quick-tap-ms`: Adjust for repeated key behavior

### Power Management

To extend battery life:
- Reduce `CONFIG_ZMK_IDLE_TIMEOUT`
- Decrease `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT`
- Keep RGB underglow disabled (currently the case)

### Enabling RGB Underglow

Uncomment lines 34-41 in `config/corne.conf`:
```
CONFIG_ZMK_RGB_UNDERGLOW=y
CONFIG_WS2812_STRIP=y
CONFIG_ZMK_RGB_UNDERGLOW_EFF_START=0
CONFIG_ZMK_RGB_UNDERGLOW_BRT_MAX=30
CONFIG_ZMK_RGB_UNDERGLOW_EXT_POWER=n
CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=y
```

Note: This will reduce battery life significantly.

## Recent Changes (from Git History)

- **Latest**: Reverted "Using home mod for other layers" (d303dd1)
- Enabled mouse simulation (b26b7a6)
- Experimented with home row mods on multiple layers (6463f5c, reverted)
- Optimized home row mod configurations for base layer (7906df5)
- Disabled RGB underglow (2557261)

## Resources

- [ZMK Documentation](https://zmk.dev/)
- [ZMK Studio Guide](https://zmk.dev/docs/features/studio)
- [Corne Keyboard Info](https://github.com/foostan/crkbd)
- [Nice!Nano Documentation](https://nicekeyboards.com/docs/nice-nano/)

## Troubleshooting

### Bluetooth Connection Issues

1. Clear bonds: Activate layer 6 and use BT_CLR
2. Re-pair the keyboard
3. Check battery level on OLED display

### Key Not Working

1. Check debounce settings in `corne.conf`
2. Test with ZMK Studio
3. Verify key matrix in device tree files

### Battery Draining Too Fast

1. Ensure RGB underglow is disabled
2. Reduce OLED brightness or disable display
3. Adjust sleep timeouts
4. Lower Bluetooth TX power

## License

This configuration is based on ZMK's MIT-licensed examples. The keymap and configuration files follow the same MIT license.
