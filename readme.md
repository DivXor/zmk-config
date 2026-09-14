# ZMK Config — Corne · Totem · Kometa

Personal [ZMK firmware](https://zmk.dev) configuration for three split keyboards,
built via GitHub Actions on every push. Layer definitions live in the
`config/*.keymap` files — this README summarizes the shared design and where
everything lives.

## Keyboards

| Board | Role | Controller | Display | Keymap |
|---|---|---|---|---|
| **Corne** (nice!view) | Daily driver | nice!nano ×2 | 2× nice!view | `config/corne.keymap` |
| **Totem** (dongle) | Secondary | XIAO BLE (dongle + halves) | Prospector on dongle | `config/totem.keymap` |
| **Kometa** | Secondary | nice!nano ×2 | none (RGB underglow) | `config/kometa.keymap` |

## Shared design language

All three keymaps share the same building blocks, so muscle memory transfers
between boards:

- **Positional homerow mods** (`config/includes/behaviours_homerow_mods.dtsi`) —
  hold home-row keys for modifiers, tuned so fast rolls ("ne", "sd") never
  false-trigger:
  - Left: `A`=Shift, `S`=Ctrl, `D`=Alt, `F`=Gui
  - Right: `J`=Gui, `K`=Alt, `L`=Ctrl, `;`=Shift
- **Combos** (same gestures everywhere):

  | Combo | Action |
  |---|---|
  | `Q`+`W` | Escape |
  | `J`+`K` | Escape |
  | `D`+`F` | Delete |
  | `A`+`S`+`D` | Enter |
  | `D`+`F`+`J`+`K` | Caps Word |
  | `X`+`C` | Copy |
  | `C`+`V` | Paste |
  | `X`+`V` | Cut |

- **ZMK Studio** — live-tune the keymap over USB from
  <https://studio.zmk.fun> without reflashing — enabled on every central
  build (corne/kometa left halves, totem dongle, corne dongle).
- Key-position labels via [`zmk-helpers`](https://github.com/urob/zmk-helpers)
  (corne uses the generic 42-key header, totem has a dedicated one, kometa uses
  numeric positions from its shield definition).

## Corne (primary)

Base layer:

```
╭──────────────────────────────────┬──────────────────────────────────╮
│ ESC    Q    W    E    R    T     │ Y    U    I    O    P    BKSP   │
│ TAB    A    S    D    F    G     │ H    J    K    L    ;    '      │
│ SHFT   Z    X    C    V    B     │ N    M    ,    .    /    SHFT   │
│           GUI  LWR  ENT/ALT      │ SPC/HYPR  RSE  RALT              │
╰──────────────────────────────────┴──────────────────────────────────╯
```

- Hold `A S D F` / `J K L ;` for Shift/Ctrl/Alt/Gui (see table above).
- `LOWER` + `RAISE` together → **ADJUST** (tri-layer).
- **LOWER**: numbers + symbols, mirrored around the space column.
- **RAISE**: F1–F12 (`ESC`-position = `F11`, `BKSP`-position = `F12`), arrows on
  right home row, screen brightness on `;`/`'`.
- **ADJUST**: output selection, media keys, Bluetooth profiles 0–4,
  `RESET` / `BOOT` (right top row) and ZMK Studio unlock — no need to open the
  case to reset the MCU.
- The `HYPER` hold (`LC(LS(LA(LGUI))`) sits on the space thumb and the `'` key.
- nice!view display enabled (`CONFIG_ZMK_DISPLAY=y`); layers carry
  `display-name`s so the screens show the active layer.
- **Screens flash on updates** — nice!view is a memory-in-pixel panel, so
  every redraw is a full-frame update with a brief visible flash: the left
  screen flashes about once per second while typing (WPM graph), and every
  layer switch / battery-percent step flashes both screens. That's inherent
  to the panel type, not a fault.
- **Battery tuning** (`config/corne.conf`): display updates pause after 30 s
  idle (the MIP keeps showing the last frame) and the halves deep-sleep after
  30 min idle — a keypress wakes them; Bluetooth reconnects within a few
  seconds and unsaved Studio RAM changes are lost. Without this, ZMK's 10 ms
  display tick runs ~100 CPU wakeups/second forever even when idle, which is
  what drained the batteries. Both timeouts are tunable in the conf file.

## Totem

Dongle setup: the XIAO BLE dongle acts as central; both halves are
peripherals. Pair left half first, then right, for correct battery order.

- Layers: **Base → Num** (numpad-style numbers), **Media** (symbols + media),
  **Function** (F-keys, Bluetooth, window management, utilities).
- Prospector display on the dongle (operator status screen, fixed brightness).
- `REGION_CAPTURE` (Ctrl+Menu) and `STUDIO` unlock keys live on the Function
  layer; `TO_0` on the Media layer returns to Base.
- See the ASCII diagrams in `config/totem.keymap` for full layouts.

## Kometa

- Same homerow mods and combos as corne/totem. Copy/paste/cut were moved from
  double-tap `C`/`V` tap-dances to combos (no more accidental Ctrl+C when
  typing "success").
- `LOWER` = numpad-style numbers/symbols; `RAISE` = F-keys, media, brightness;
  **ADJUST** (via `ADJ` layer-tap thumbs) = Bluetooth, RGB underglow, soft-off,
  reset/bootloader, studio unlock.
- `RGUI`+`Space` (right pinky) opens Spotlight on macOS.
- Soft-off is enabled (`CONFIG_ZMK_PM_SOFT_OFF=y`): hold the reset button for a
  few seconds to fully power the board down.

## Dongle builds

The XIAO dongle hardware can serve as a USB receiver for all keyboards, not
just the totem. Two extra dongle firmwares are built for it:

| Firmware | Drives | Config switching | Peripheral budget |
|---|---|---|---|
| `totem-dongle` | totem | fixed keymap | 2 |
| `corne-dongle` | corne + kometa | fixed keymap (shared 42 positions) | 4 |

Dongles never deep sleep — they're USB-powered and always connected, and a
sleeping central couldn't be woken by the halves' keys anyway (the dongle has
no key matrix). `config/corne_dongle.conf` and `config/totem_dongle.conf` pin
this down against the shared `corne.conf`/`totem.conf` settings, which leak
into the dongle builds through the shield-name resolution. The battery-driven
halves still sleep on their own timers (30 min idle on corne).

### Corne + kometa through a dongle

Corne and kometa share the identical 42-position matrix, so `corne-dongle`
(its keymap is `corne.keymap`) drives either board's halves: flash the
`corne-dongle-left/right` and `kometa-dongle-left/right` artifacts onto the
halves (peripheral mode), bond them to the dongle, and power on whichever
board you want. Note kometa's outer columns act as corne keys through the
dongle (`[`/`]`/`\`/grave become ESC/BSPC/TAB/shift).

### Bluetooth host output (optional)

Each dongle is a normal ZMK central, so the keymap's output keys drive it
too: use corne **ADJUST**'s `OUT_B`/`OUT_U` (plus `BT1`/`BT2`) or the totem
**FN** layer's output/BT keys to switch the dongle between USB and Bluetooth
hosts, and pair it with several hosts just like a standalone keyboard.

- Host profile budget: pairing slots are shared between halves and hosts —
  `corne-dongle` leaves **2** host profiles (6 slots − 4 halves),
  `totem-dongle` leaves **4**. Profile slots beyond the budget are inert.
- **Halves' battery is unaffected**: each half maintains exactly one BLE
  link — to the dongle — no matter how the dongle reaches the host. Only
  the dongle's own battery drains faster while a Bluetooth host is active.
- Switching back with `OUT_U` (or unplugging/`OUT_B` cycling) restores USB
  as the output; nothing needs reflashing.

### Pairing halves to a dongle

Bonds live in persistent settings storage and **survive reflashing**, and a
bonded peripheral directed-advertises *only to its old central* — so stale
bonds make a half invisible to any other dongle. Whenever you introduce a
central/half combination that hasn't been bonded before:

1. Flash the `settings_reset` artifact onto the **half** to wipe its bonds,
   then flash the peripheral firmware you actually want on it.
2. Flash `settings_reset` onto the **dongle** as well when you move it
   between keyboard families (e.g. it previously lived as a totem dongle):
   its six-device pairing pool still holds every old bond, which can block
   new halves from pairing.
3. Plug in the dongle, power the halves: unbonded peripherals advertise
   automatically and the dongle connects as they appear. Introduce halves
   one at a time; pairing left first keeps battery reporting order correct.

Clearing the dongle wipes its split bonds too, so halves that were paired
to it before (e.g. totem halves and a repurposed dongle) must be reset and
re-paired as well.

## Building & flashing

1. Push (or open a PR) — GitHub Actions builds all firmware automatically.
2. Download the artifact for your half from the **Actions** tab
   (`corne_left`, `corne_right`, `kometa_left`, `kometa_right`,
   `totem-dongle`, `totem-dongle-left`, `totem-dongle-right`,
   `corne-dongle`, `corne-dongle-left/right`, `kometa-dongle-left/right`).
3. nice!nano / XIAO: double-tap the reset button, then drag the `.uf2` file
   onto the mounted drive.

Settings-reset images (`xiao_reset_settings`, `nice_nano_reset_settings`) are
also built — flash one onto each half if Bluetooth profiles get stuck, then
re-pair.

## ZMK Studio

All boards ship with ZMK Studio enabled on their central half. Connect the
central over USB and open <https://studio.zmk.fun> (Chrome/Edge, WebUSB) to edit
the keymap live. The keymap is locked at boot; unlock with:

- **Corne**: ADJUST layer (hold LOWER + RAISE) → third key of the right top row.
- **Kometa**: ADJUST layer (via the `ADJ` layer-tap thumb keys on LOWER/RAISE) → middle thumb keys.
- **Totem**: Function layer → `STUDIO` key.

Changes made in Studio live in RAM only — copy anything you like back into the
`.keymap` files and commit.

## Bluetooth

Each keyboard stores 5 profiles (`BT_SEL 0`–`BT_SEL 4`). To pair a new device,
select an empty profile, then hold the pair button / trigger pairing from the
host. Use `BT_NXT`/`BT_PRV` on corne, or the `BT_SEL n` keys on each board's
utility layer, to switch hosts.

## Repo layout

```
build.yaml                    # per-shield build matrix (incl. dongle variants)
config/
├── corne.keymap|conf         # primary; also serves corne-dongle
├── totem.keymap|conf         # + boards/shields/totem (dongle shield)
├── kometa.keymap|conf        # + boards/shields/kometa (custom shield)
├── boards/shields/corne_dongle/       # keyless central shield (corne/kometa)
├── includes/
│   ├── behaviours_homerow_mods.dtsi   # shared positional homerow mods
│   └── combos.dtsi                     # shared combos (totem & corne labels)
└── west.yml                  # zmk/zmk-helpers/prospector pinned to commit SHAs
```

ZMK, zmk-helpers, and the prospector module are pinned to exact commit SHAs in
`west.yml` (and the reusable workflow is pinned to the same ZMK commit).
Note: ZMK's release tags can't be used here — the last release (v0.3.0)
predates the HWMv2 board names (`nice_nano//zmk`, `xiao_ble//zmk`) this
config uses. Bump all SHAs together, deliberately, when you want newer
firmware.