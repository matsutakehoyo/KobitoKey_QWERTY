# KobitoKey — Final Build Notes and Known Quirks

Drafted: 2026-09-25 (covers the September 2026 firmware/hardware finalisation)

Final firmware: **`6867c12`** (KobitoKey fork) + **`c3f3dbc`** (zmk-naginata fork)
— flashed to both halves 2026-09-25, verified working.
Artifacts archived: `firmware/custom/6867c12/firmware/`

## 1. Final configuration

- **Repos**
  - Keymap/config: https://github.com/matsutakehoyo/KobitoKey_QWERTY (fork of juichi50iii, from `v1.0.1`)
  - Naginata module: https://github.com/matsutakehoyo/zmk-naginata (fork of eswai, pinned in `config/west.yml`)
- **Layers**: QWERTY (home-row mods ⇧⌃⌥⌘) / NAGINATA / LOWER / MOUSE / RAISE / FUNCTION / SHORTCUT.
  ASCII diagrams + key-position matrix live as comments in `config/KobitoKey.keymap`.
- **Language switching** (matches old QMK Arasaa muscle memory):
  - `M+,` together (layer 0 combo, or alone in kana mode) → kana on
  - `C+V` pressed and released alone in kana mode → kana off
  - 英数/かな thumb keys switch IME **and** layer together (`&ng_off`/`&ng_on` macros)
- **Naginata extras**
  - Editing modes: hold `D+F` / `J+K` / `C+V` / `M+,` + key (cursor, selection, clipboard, Del…)
  - カタカナ/ひらがな変換: `D+F+;` / `D+F+/` (IME shortcuts, OS-aware); 再変換: `D+F+I` or `C+V+I`
  - **Roman passthrough**: hold either outer thumb Shift on the naginata layer → letters
    bypass kana conversion (capitals, e.g. acronyms). Works with any modifier (⌘/⌃/⌥ shortcuts too).
- **Module fork changes** (why a fork exists at all):
  1. On/off gestures moved into the engine dictionary (M+,/C+V) with deferred judging,
     so C+V-held still enters editing mode — a ZMK position combo cannot do this
     (it fires before the behavior sees the keys; this was the original editing-mode bug).
  2. `naginata_on()/naginata_off()` switch the ZMK layer themselves
     (`CONFIG_NAGINATA_LAYER`, default 1) — IME and layer can no longer desync.
  3. Modifier passthrough (QMK `process_modifier` equivalent).

## 2. Build & flash workflow

- Build: pushes do **not** trigger CI on the fork — dispatch manually:
  `gh workflow run build.yml --repo matsutakehoyo/KobitoKey_QWERTY --ref main`,
  then `gh run download <id> --dir firmware/custom/<shortsha>`.
- Flash: double-tap reset → half mounts as `XIAO-SENSE` → copy matching UF2.
  - macOS prints `could not copy extended attributes … Operation not permitted`
    (sometimes `Device not configured`) — **harmless**; the volume unmounting itself
    means the flash was accepted.
  - Keymap-only changes don't need `settings_reset`; BLE pairing survives.
- GitHub CI artifacts expire (~90 days); the durable archive is `firmware/custom/` in Dropbox.

## 3. Known quirks

- **Sustained-overlap on/off gestures**: rolling はこ (C+V) or なん (M+,) with ≥50 ms
  overlap triggers off/on instead of kana — same behaviour as the QMK Arasaa;
  the 50 ms overlap-split (`CONFIG_NAGINATA_MIN_OVERLAP_MS`) protects normal rollover.
- **Editing-mode Unicode symbols** (「」『』？！○《》…) in the J+K / M+, blocks need the
  Ishizuki helper on macOS. Deliberately **not used** — keyboard must work on any machine
  with zero host setup. Full-width symbols come from LOWER/RAISE through the IME instead
  (`?` was added to RAISE next to `!` for this). Cursor/clipboard editing functions are
  plain keycodes and work everywhere.
- **Do not add ZMK combos on layer-1 edit-shift pairs** (D+F, J+K, C+V, M+,) — breaks
  editing mode silently (see §1). The only naginata combo is M+, on layer 0.
- **Left trackball** is scroll (horizontal enabled via `zip_x_scaler 1 15` in the left
  overlay); right trackball is pointer with auto mouse layer.
- **LED layer colours** configured up to layer 4 in `KobitoKey_left.conf`.

## 4. Hardware repair log

- **2026-09-09 — K key (right half) dead**: intermittent, then fully dead on all layers.
  Isolated by switch-swap + tweezer test → cold solder joint on the K-position diode.
  Reflowed; verified. Lesson: single dead key with working neighbours = switch pin,
  socket, or diode joint — matrix/controller/BLE would take out more than one key.

## 5. September commit trail (since `19bfc3e`)

| Commit | Change |
| --- | --- |
| `0345030` | Fixed naginata editing mode & on/off sync (combo positions, layer-aware lang keys) |
| `21a7cad` | Added `?` to raise layer next to `!` |
| `0b63cdc` | Layer diagrams as keymap comments (Arasaa style) |
| `1fd2b53` | Key-position matrix comment |
| `a8196e1` | Switched to forked zmk-naginata (M+,/C+V, module layer switching) |
| `4798ec6` | README: fork documentation |
| `6867c12` | Naginata-layer Alt thumbs → Shift (roman passthrough) |

zmk-naginata fork: `149ab91` (M+,/C+V + layer switching), `c3f3dbc` (modifier passthrough).
