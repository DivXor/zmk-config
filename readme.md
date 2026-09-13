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

- **ZMK Studio** enabled on every board — live-tune the keymap over USB from
  <https://studio.zmk.fun> without reflashing (see below).
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
| `universal-dongle` | all three | **Z+X combo** swaps between configs | 6 |

### Corne + kometa through a dongle

Corne and kometa share the identical 42-position matrix, so `corne-dongle`
(its keymap is `corne.keymap`) drives either board's halves: flash the
`corne-dongle-left/right` and `kometa-dongle-left/right` artifacts onto the
halves (peripheral mode), bond them to the dongle, and power on whichever
board you want. Note kometa's outer columns act as corne keys through the
dongle (`[`/`]`/`\`/grave become ESC/BSPC/TAB/shift).

### Universal dongle

`universal-dongle` carries both configurations in one firmware (`config/
universal_dongle.keymap`): totem as layers 0–3, corne/kometa as layers 4–7.
Press **Z+X** — the same physical gesture on every board — to toggle between
them (prospector displays "Totem…"/"Corne…" layer names). Caveats:

- The dongle boots into the totem config (layer 0); press Z+X once after
  replug to switch.
- Only power the keyboard matching the active config — keys of the "wrong"
  board arrive scrambled.
- Six bonded peripherals on one central is untested territory; verify it
  works with your hardware.

### Pairing halves to a dongle

Halves bond to exactly **one central**. To move a half between standalone
and dongle use: flash the appropriate firmware, flash the `settings_reset`
image onto the half to clear old bonds, then pair it to the dongle (peripherals
that are not bonded advertise automatically — the dongle connects as they
wake). Pairing left first keeps battery reporting order correct.

## Building & flashing

1. Push (or open a PR) — GitHub Actions builds all firmware automatically.
2. Download the artifact for your half from the **Actions** tab
   (`corne_left`, `corne_right`, `kometa_left`, `kometa_right`,
   `totem-dongle`, `totem-dongle-left`, `totem-dongle-right`,
   `corne-dongle`, `corne-dongle-left/right`, `kometa-dongle-left/right`,
   `universal-dongle`).
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
├── universal_dongle.keymap   # totem + corne configs for universal-dongle
├── boards/shields/corne_dongle/       # keyless central shield (corne/kometa)
├── boards/shields/universal_dongle/   # keyless central shield (all boards)
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