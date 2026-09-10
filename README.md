# KobitoKey_QWERTY (matsutakehoyo fork)

Personal ZMK firmware for my KobitoKey, forked from
[juichi50iii/KobitoKey_QWERTY](https://github.com/juichi50iii/KobitoKey_QWERTY) `v1.0.1`.
macOS / Japanese workflow: QWERTY with home-row mods + 薙刀式 kana input.

## Layers

ASCII diagrams for every layer (and a key-position matrix for combos) are
maintained as comments inside [`config/KobitoKey.keymap`](config/KobitoKey.keymap).

| # | Layer | Notes |
|---|-------|-------|
| 0 | QWERTY | home-row mods ⇧⌃⌥⌘ / ⌘⌥⌃⇧, dedicated 英数/かな thumbs |
| 1 | NAGINATA | 薙刀式 kana via forked zmk-naginata module |
| 2 | LOWER | numbers, brackets, quotes |
| 3 | MOUSE | auto mouse layer, buttons on right hand |
| 4 | RAISE | symbols (incl. `?` next to `!`) + navigation |
| 5 | FUNCTION | Bluetooth profiles + F-keys (LOWER+RAISE) |
| 6 | SHORTCUT | macOS app/window switching, clipboard, paste-and-match |

## 薙刀式 (Naginata) design

Kana input uses my fork of eswai's module:
[matsutakehoyo/zmk-naginata](https://github.com/matsutakehoyo/zmk-naginata)
(pinned in [`config/west.yml`](config/west.yml)). The fork makes two changes
to match my QMK Arasaa setup (`naginata_v15.c`):

- **On/off gestures are engine dictionary entries, not ZMK combos**:
  **M+, = kana on**, **C+V = kana off**. The engine's deferred simultaneous-press
  judging distinguishes *C+V pressed and released alone* (off) from *C+V held
  while a third key is pressed* (editing mode 2), exactly like QMK.
- **`naginata_on()/naginata_off()` switch the ZMK layer themselves**
  (`CONFIG_NAGINATA_LAYER`, default 1), so the IME state and the layer can
  never desync.

⚠️ **Do not place ZMK combos on naginata edit-shift key pairs (D+F, J+K, C+V,
M+,) on layer 1.** A position combo fires before the naginata behavior sees
the keys and silently breaks editing mode — this was the original bug that
motivated the fork. The only naginata-related combo left is M+, on **layer 0**
(the module never sees layer-0 keys, so entering kana needs a combo there).

Other entry/exit paths: the かな thumb key runs `&ng_on`, the 英数 thumb key
on the naginata layer runs `&ng_off` (both macros send the IME keycodes and
switch layer).

**Editing-mode symbols** (「」『』？！ etc.) are emitted as Unicode hex input
and would require the [Ishizuki](https://github.com/eswai/Ishizuki) helper on
macOS. I deliberately don't use it — the keyboard should work on any machine
without host-side setup — so full-width symbols come from the LOWER/RAISE
layers through the IME instead. The cursor/clipboard editing functions
(D+F / J+K / C+V / M+, held) are plain keycodes and work everywhere.

## Building

GitHub Actions builds the firmware, but **a push does not trigger it** — 
dispatch manually:

```sh
gh workflow run build.yml --repo matsutakehoyo/KobitoKey_QWERTY --ref main
gh run download <run-id> --repo matsutakehoyo/KobitoKey_QWERTY --dir firmware/custom/<shortsha>
```

## Flashing

Double-tap reset on a half → it mounts as `XIAO-SENSE` → copy the matching
UF2 (`KobitoKey_left …` / `KobitoKey_right …`). macOS prints a harmless
"could not copy extended attributes" warning; the drive unmounting itself
means the flash was accepted. Keymap-only changes don't need
`settings_reset` — pairing survives.

---

## Upstream README

小人キーや人キーのケース、TypeSurfer各種の3DデータはReleasesよりダウンロード出来ます。
ダサい使い方はしないこと。

Layer 0 QWERTY
<img width="1280" height="690" alt="Image" src="https://github.com/user-attachments/assets/ef0797b7-a63f-4632-912d-9b5d0115769f" />

Layer 1 NUMBER & ARROW
<img width="1280" height="690" alt="Image" src="https://github.com/user-attachments/assets/d6347b3c-a238-4278-bacd-e58195774d0e" />

Layer 2 Bluetooth & FUNCTION
<img width="1280" height="690" alt="Image" src="https://github.com/user-attachments/assets/f1f7cc93-fbd8-4a98-84ea-c8c36ad3952d" />

Layer 3 AUTO MOUSE
<img width="1280" height="690" alt="Image" src="https://github.com/user-attachments/assets/2efe5275-e460-41bc-ae45-0c0665435268" />
