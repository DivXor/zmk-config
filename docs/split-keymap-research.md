# Split-keyboard layer design: research & reference survey

Scope: layer architecture for small split boards (34–42 keys), **not** alpha
layouts (base stays QWERTY). Sources are primary configs/write-ups by the
people who built the designs, plus one quantitative symbol-frequency study.
Reddit could not be fetched from this network (JSON API blocked); the designs
below are the ones those threads consistently converge on, which is why they
were fetched at the source instead.

Survey done 2026-02 in preparation for the keymap unification (totem-target
design, shared across corne/kometa/totem). Constraints already settled:
QWERTY base, tuned positional HRM kept, numpad-style numbers (no `KP_*`),
arrows on a home row, easy corne→totem migration.

---

## 1. The design families

### 1.1 Miryoku (manna-harbour) — "orthogonal sublayers"

Source: <https://github.com/manna-harbour/miryoku> (reference manual in
`docs/reference/readme.org`).

- 6 content layers: `NAV`, `MOUSE`, `MEDIA` (right-hand layers) and `NUM`,
  `SYM`, `FUN` (left-hand layers), plus `BUTTON` and base.
- **Layer key on the thumb of the *opposite* hand** from the layer's content:
  hold left home thumb → NAV on the right hand. One hand holds the layer, the
  other does the work; no finger contortions.
- **All modifiers live on the home row of the same hand as the layer-hold
  thumb** — so layer-change and mods can be held in any order or
  simultaneously, with no tap/hold race conditions. Any mod + any key is a
  two-hand operation with zero timing.
- Every layer has a **single purpose per hand**; all layers share the same
  basic arrangement (fun mirrors num, media mirrors nav), which halves the
  learning cost.
- Thumbs on base: `BSPC / RET / DEL` right, `SPC / TAB / ESC` left; each thumb
  key is a dual-function tap/layer-hold (tap-dance for sublayers).
- No home-row mods at all — mods only live on layers.

**Arguments for:** deterministic (no tap/hold decisions outside thumbs),
fully symmetric, scales down to 36 keys cleanly, mods-chord problem
structurally solved.
**Arguments against:** two-hand operation for everything (can't mod+navigate
one-handed); thumbs do heavy lifting; small mods like "hold ctrl while
clicking mouse" need the button layer.

### 1.2 Callum Oakley — "half-layer mods"

Source: <https://github.com/callum-oakley/keymap> (34-key Ferris).

- Only 2 content layers: `nav` and `sym`, **each available as a half-layer on
  *both* thumbs** — left thumb activates the left half of the layer, right
  thumb the right half.
- Each half carries its own home-row real modifiers (`cmd alt shift ctrl esc`
  in a cross around home position). The half you are **not** holding stays
  transparent, so you can always mod keys on the *other* half: `cmd+c` =
  hold right-sym + right-cmd + `c` (base `c` stays visible).
- Numbers live inside the `sym` layer (1–5 left, 6–0 right, on the top two
  rows); `- ` = [ ] ; \ ` around them.
- Callum explicitly rejected: mod-taps (timing issues), chords/combos (same
  timing issues), and his own one-shot "callum mods" (two distinct movements
  per shortcut, must re-arm mods between presses). Plain layers + real mods
  won.
- `fn` layer = nav-hold + right-sym-hold (both thumbs) — i.e. a *two-thumb
  tri-layer*.

**Arguments for:** simplest possible mental model; modding anything with
anything (base or layer keys) needs no extra mechanism; numbers+symbols in one
layer = fewer layer switches.
**Arguments against:** typing an uppercase symbol (`&` = shift+7) takes four
key-presses in two rolls (both thumbs + shift + key) — callum argues it's one
fluid motion; 34-key sym layer is dense; only one symbol layer so no
numpad-style number block.

### 1.3 urob — "combos + smart layers on top of HRM"

Source: <https://github.com/urob/zmk-config> (README is a full design essay;
keymap in `config/base.keymap`).

- 36-key Corne-ish zen. Base QWERTY-family (Colemak-DH) with his "timeless"
  positional HRM (the same `require-prior-idle-ms` + `hold-trigger-key-positions`
  scheme this repo already uses, adapted from it).
- Thumbs: `space`(hold → NAV), `smart-num` (tap = numword, double-tap =
  sticky num layer, hold = num layer), FN (hold) + `RET`, magic-shift thumb.
- **Symbols mostly via combos instead of a layer**: vertical combos on the
  number-row positions reproduce the shifted number row (`!@#$%...`), symmetric
  bottom combos (`_^`, `-+`, `/*`, `|&`), horizontal bracket combos. Prefers
  combos over lateral thumb layer-switches in fast succession; relies on
  `require-prior-idle-ms` to kill misfires.
- NAV layer: `&sk` sticky one-shot mods on the left home row + nav cluster on
  the right with **hold-taps on the nav cluster** (`home/end`, doc
  begin/end, delete word) — more functions than keys.
- FN layer: F-keys with HRM *on the F-keys themselves* (`hml LGUI F11`…),
  desktop-management macros, media keys.
- NUM layer: numpad block (7-8-9 / 4-5-6 / 1-2-3, `0` on home) with HRM
  overlaid on the digits.
- SYS layer via **conditional tri-layer** (FN+NUM → SYS: BT profiles, reset,
  bootloader).
- `num-word` auto-layer (numbers layer auto-exits on the first non-number
  key) via his `zmk-auto-layer` module; `smart-mouse` similarly self-exits.

**Arguments for:** richest ergonomics-per-key budget; auto-exiting layers
remove most layer-toggle cognitive cost; combos give one-keystroke symbols
with no layer at all.
**Arguments against:** deepest complexity of any public config (custom
modules, tap-dances, mod-morphs, macros); much of the machinery assumes his
exact base layout; auto-layers have corner cases (numbers followed by the
letters under the numpad don't auto-exit).

### 1.4 Florian Gächter (flo) — "every thumb is a layer-tap"

Source: <https://flo.gaechter.xyz/posts/totem-keyboard/> + config
<https://github.com/floriangaechter/zmk-config> (`config/totem.keymap`).

- Totem, QWERTY base, classic `&mt` HRM, `HYPER` on the left wing key.
- **All six thumbs are layer-taps**: `ESC/MED · SPACE/NAV · TAB ‖ RET/SYM ·
  BSPC/NUM · DEL/FUN`. Five momentary layers, one per thumb hold.
- NAV: left hand = real `LGUI LALT LCTRL LSHFT` home-row mods; right hand =
  arrows + clipboard + ins/pgup/pgdn/home/end. (Miryoku-style "mods on the
  free hand".)
- NUM: numpad 789/456/123 with `[ ] ; = ` \ . 0 -` wrapped around it; right
  home row = real `RSHFT RCTRL RALT RGUI`.
- SYM: `& * ( …` top, `$ % ^ +` home, `! @ # |` bottom — shifted-number-row
  symbols grouped mnemonically; right home row = real mods again.
- FUN: F1–F12 in numpad-mirroring positions + printscreen; media on MED; BT +
  mouse buttons on a separate `BUT` layer.

**Arguments for:** same single idea everywhere (every layer reachable from a
thumb, every free hand has real mods); nothing to memorize beyond "hold a
thumb"; directly proven on totem.
**Arguments against:** ESC/DEL on outer thumbs is a long reach on some
boards; media/BT layers exist just to fill thumb holds; `&mt` tap-preferred
HRM is dated compared to timeless HRM.

### 1.5 Seniply (stevep99) — "six layers, zero mod-taps, sticky mods"

Source: <https://stevep99.github.io/seniply/> (Colemak-DH by default, works
with QWERTY base too).

- 34-key minimum. **No dual-role keys at all**; all six layers on thumb holds:
  Shift (inner-left), Extend (outer-left), Sym (outer-right), Space
  (inner-right), Num & Fun via two-thumb chords or spare keys.
- Sticky (one-shot) modifiers on the Extend/Sym/Num layers' left home row:
  tap ctrl, tap alt, then the key — no holds required, no timing, order-free.
- Extend layer (DreymaR-derived): nav right, one-shot mods left, BSPC/DEL on
  strong index/home positions, cut/copy/paste/undo dedicated.
- Sym layer: brackets get one finger each (open bracket on home — IDEs
  auto-close), shifted-number-row symbols in standard order on the left.
- Num layer: numpad-style on the right hand, math symbols around it, sticky
  mods on the left home row.

**Arguments for:** zero tap/hold ambiguity anywhere (thumbs are pure holds,
mods are one-shots); very forgiving; "movements larger than 1 key-unit are
avoided".
**Arguments against:** one-shot mods are sequential (tap-tap-key) rather
than a simultaneous chord; two-thumb layer chords (Num/Fun) are awkward
mid-flow.

### 1.6 Symbol-layer engineering (getreuer + ShelZuuz + Sunaku)

Source: Pascal Getreuer, *Designing a Symbol Layer* (2021, updated 2025):
<https://getreuer.info/posts/keyboards/symbol-layer/index.html>. This is the
best quantitative treatment of symbol placement for programmers.

Design principles distilled:

1. Most-used keys on home row, rare keys in corners.
2. Avoid pinkies for often-doubled symbols (`==`, `++`, `//`); put
   non-doubled symbols (`! ~ \` → " ') on pinkies. (Or use a repeat key.)
3. Make common code bigrams **inward rolls** (pinky→index on one hand):
   `!=`, `<=`, `+=`, `->`, `()`, `[]`, `{}`, `~/`, `();`.
4. Prefer learnability: keep mnemonics (shifted-number-row order), keep
   `, .` at base positions, group related symbols.

Symbol frequency in code (top-10 per corpus, from getreuer's counts):

| Rank | C/C++ | Python | Shell |
|---|---|---|---|
| 1 | `_` | `_` | `"` |
| 2 | `*` | `.` | `-` |
| 3 | `,` | `,` | `$` |
| 4 | `)` | `)` | `0` |
| 5 | `(` | `(` | `=` |
| 6 | `.` | `'` | `1` |
| 7 | `/` | `"` | `_` |
| 8 | `0` | `=` | `/` |
| 9 | `;` | `0` | `]` |
| 10 | `-` | `:` | `[` |

Key takeaways: `_ . , ( )` dominate code — they should be the cheapest
symbols you have. `;` and `:` are cheap and frequent. `< >` are rare despite
living on Shift+,/. on stock boards. Digits 0-2 beat most symbols (Benford);
getreuer even argues digits belong on base, though that doesn't apply at
34-42 keys. Comment characters of your languages (`#`, `//`, `/* */`) are
high-frequency.

Other surveyed symbol designs: ShelZuuz (C++-optimized 3×5, layer-taps on
ring fingers, `();` as outward roll, `!=` inward), Sunaku (9 years of
Vim-optimized iterations, "crown jewel"), Dusty Pomerleau (Elixir
`|> <> -> <-` rolls), BEAKL 15, Jonas Hietala's T-34 (Rust/Elixir, symbols
split across numpad+symbol layers).

### 1.7 Ergonomics / HRM references

- **Precondition's HRM guide** (via thomasbaart.nl, 2024): canonical treatment
  of HRM flavors, timing, misfires; endorses per-hand/positional variants.
- **urob's timeless HRM** (README §Timeless homerow mods; also the top-voted
  r/ErgoMechKeyboards advice threads, e.g. "LPT: try urob's timeless homerow
  mods", "New ZMK features, better home-row mods"): `require-prior-idle-ms` +
  `hold-trigger-key-positions` + `hold-trigger-on-release`. This repo
  already implements it.
- **Known hard limit (SO 79241474, confirmed by the above)**: simultaneous
  same-side HRM chords (pressing J+K+L at once expecting three mods) degrade
  to taps, because one chord key's press resets `require-prior-idle-ms` for
  the next. Not tunable away. Fix = real mods on a layer (miryoku/flo/callum)
  or one-shots (seniply/urob-nav).
- **Miryoku principles**: use layers instead of reaching; single purpose per
  hand per layer; mods on the layer-holding hand; auto-repeat preserved on
  the working hand.
- **urob on combos vs layers**: combos beat lateral thumb layer-switches for
  fast one-off symbols; `require-prior-idle-ms` makes home-row combos safe.

---

## 2. Synthesis for this repo

Facts that drive the variant design:

1. The **multi-modifier chord problem** (IntelliJ) is solved three ways in the
   wild: real mods on the free hand (miryoku/flo/callum), one-shot sticky
   mods (seniply/urob), or a hyper alias. Real mods are the only
   *simultaneous* solution; one-shots are sequential but race-free.
2. **Numpad-style numbers** are the community default (miryoku, flo,
   seniply, urob, this repo's totem); nobody keeps a two-row number layer.
3. **Symbols**: either a dedicated SYM layer (miryoku/flo/getreuer), symbols
   wrapped around the numpad (corne default, flo's brackets, T-34), or
   combos instead of a layer (urob). Frequency data favors home-row
   `_ ( ) . , ; : =`.
4. **Tri-layer / two-hold layers** are standard for BT/reset/sys keys
   (urob's conditional layer, miryoku's two-thumb chords, callum's fn).
5. **Thumb philosophy** is the main axis of disagreement: all-layer-tap thumbs
   (flo) vs. few thumb holds + combos (urob) vs. pure holds, no dual-role
   anywhere (seniply/miryoku).
6. Corne has 6 thumbs + 4 corner keys of slack; totem has 6 thumbs + 2 wing
   keys. Any design must fit **6 thumb-holds max** and keep RET-left-inner /
   SPC-right-inner (settled).

The three variants built from this survey are documented in the session
notes; each maps to a branch for live testing.