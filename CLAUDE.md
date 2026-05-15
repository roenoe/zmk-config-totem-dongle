# TOTEM Dongle ZMK Config

Mechboards pre-build TOTEM split keyboard (Seeeduino XIAO BLE nRF52840 on both halves) with a second XIAO BLE as a USB dongle. The dongle acts as BLE central connecting to both halves and presents as USB HID — avoids BT pairing issues across reboots/sleep on the host.

## Environment (Arch Linux)

- West v1.5.0 via pip
- `protobuf` and `grpcio-tools` via pip — required for nanopb when building with ZMK Studio
- Zephyr SDK 0.17.0 at `~/tmp/zephyr-sdk-0.17.0` (ARM only, registered via `./setup.sh -t arm-zephyr-eabi -c`)
- Repo cloned to `~/src/zmk-config-totem-dongle`
- `west init -l config && west update` to initialize

## CRITICAL: Board qualifier

Always use `-b xiao_ble/nrf52840/zmk` — NOT `-b xiao_ble` or `-b seeeduino_xiao_ble`. The `zmk` variant enables NVS, FLASH_MAP, and other required settings. Without it, `CONFIG_SETTINGS_NONE` is used — BLE bonding fails, ZMK Studio saves fail, nothing persists.

## Build

Run as a single line (zsh treats `\` continuations as separate commands when pasted interactively). Always `rm -rf build/<target>` before reconfiguring to avoid stale cmake cache.

```sh
west build -d build/dongle -s zmk/app -b xiao_ble/nrf52840/zmk -- -DZephyr_DIR=/home/roenoe/src/zmk-config-totem-dongle/zephyr/share/zephyr-package/cmake -DSHIELD=totem_dongle -DZMK_CONFIG=/home/roenoe/src/zmk-config-totem-dongle/config
```

Shields: `totem_left`, `totem_right`, `totem_dongle`, `settings_reset` — all use `xiao_ble/nrf52840/zmk`.

Flash: double-tap reset to enter bootloader, then `cp build/dongle/zephyr/zmk.uf2 /run/media/roenoe/XIAO-SENSE/`.

## GitHub Actions

Builds run automatically on push via `.github/workflows/build.yml` (calls zmkfirmware reusable workflow). Artifacts (`.uf2` files) are downloadable from the Actions tab on GitHub — click a completed run, then download under "Artifacts". `build.yaml` lists all four shields with `board: xiao_ble/nrf52840/zmk`.

## ZMK Studio

The `studio-rpc-usb-uart` snippet does NOT exist in this ZMK version — configured manually via `config/totem_dongle.conf` and `config/totem_dongle.overlay`. Dongle shows up as `/dev/ttyACM0`. User is in `uucp` group for access. Restart ZMK Studio after flashing new firmware before connecting.

Limitations:
- `&sk` (OSM/sticky key) bindings cannot be set via ZMK Studio UI — must be hardcoded in the keymap.
- Combos cannot be created or edited in ZMK Studio — must be defined in the keymap file.
- ZMK Studio overrides compiled key bindings at runtime (stored in NVS), but behaviors and combos are always sourced from the compiled firmware.

## Re-pairing procedure

When bonding needs to be reset (e.g. after flashing all new firmware):
1. Flash `settings_reset` to dongle, left, right
2. Flash actual firmware to dongle, left, right
3. Power on: dongle first, then left, then right

## Keymap

Custom Norwegian layout designed with cyanophage's analyzer tool. Homerow mods on home row (WIN/ALT/CTRL/SHIFT). Right-hand vowels.

```
 X  W  F  P  Z      Y  Å  O  U  Æ
 L  R  S  T  G      B  N  E  I  A
⌨  K  M  C  D  V   J  H  ,  .  Ø  Q
```

- Left outer bottom key `⌨` (position 20): hold=LCTRL, tap=ESC (`&hm LCTRL ESCAPE`)
- Right outer bottom key (position 31): Q (no special behavior)
- Left thumb (L→R): `&sl 2` (sticky Num), `SPACE`, `&mo 1` (momentary Nav)
- Right thumb (L→R): `&sk LSHFT` (one-shot shift), `BSPC`, `ENTER`

### Homerow mod assignments

Left hand: L=LWIN, R=LALT, S=LCTRL, T=LSHIFT

Right hand: N=RSHIFT, E=RCTRL, I=**LALT** (not RALT — only LALT works with user's OS shortcuts), A=**LWIN** (not RWIN)

### Dead key macros

Norwegian dead keys (¨, ^, `) cannot be output directly — pressing them once primes a dead key. Three macros in the keymap send each keycode twice to output the raw character:
- `m_umlaut`: sends `NB_UMLAUT` twice → outputs ¨
- `m_caret`: sends `NB_CARET` twice → outputs ^
- `m_grave`: sends `NB_GRAVE` twice → outputs `

## Key positions

Used for combo definitions. Positions follow reading order across each row.

```
 0   1   2   3   4       5   6   7   8   9      ← top row
10  11  12  13  14      15  16  17  18  19      ← home row
20  21  22  23  24  25  26  27  28  29  30  31  ← bottom row (20=left outer, 31=right outer)
                32  33  34  35  36  37          ← thumbs
```

## Combos

All combos use home row or thumb keys:

| Keys | Positions | Output |
|------|-----------|--------|
| T + N (index fingers) | 13 + 16 | Enter |
| S + E (middle fingers) | 12 + 17 | Escape |
| R + I (ring fingers) | 11 + 18 | Tab |
| SPACE + BSPC (thumbs) | 33 + 36 | Sticky Sym layer (`&sl 3`) |

## Layer structure

5 layers total (Media and Meta removed):

| # | Name | Access |
|---|------|--------|
| 0 | Base | default |
| 1 | Nav | `&mo 1` (right left thumb) |
| 2 | Num | `&sl 2` (left left thumb, sticky) |
| 3 | Sym | combo SPACE+BSPC (`&sl 3`, sticky) |
| 4 | Gaming | `&to 4` from Base; `&to 0` to exit |

Gaming layer is managed via ZMK Studio — not edited in the keymap file.
