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

## ZMK Studio

The `studio-rpc-usb-uart` snippet does NOT exist in this ZMK version — configured manually via `config/totem_dongle.conf` and `config/totem_dongle.overlay`. Dongle shows up as `/dev/ttyACM0`. User is in `uucp` group for access. Restart ZMK Studio after flashing new firmware before connecting.

Limitations: `&sk` (OSM/sticky key) bindings cannot be set via ZMK Studio UI — must be hardcoded in the keymap.

## Re-pairing procedure

When bonding needs to be reset (e.g. after flashing all new firmware):
1. Flash `settings_reset` to dongle, left, right
2. Flash actual firmware to dongle, left, right
3. Power on: dongle first, then left, then right

## Keymap

Semimak-ish Norwegian layout. Homerow mods on home row (WIN/ALT/CTRL/SHIFT).

```
X H F Å Ø   B P M L Z
I O E A .   G T S R N
J K Y U ,   V D C W Q
```

- Left thumb: `ESC(layer5)`, `TAB(layer1)`, `SPACE(layer4)`
- Right thumb: `OSM LSHFT (&sk LSHFT)`, `BSPC(layer2)`, `DELETE`
- Outer row 3 keys: Å (left), Æ (right)
