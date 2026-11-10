# Mindset Personal Computer (1984) — firmware

Four processors live on this board, and three of them run firmware out of
their own EPROM/mask ROM: the 80186 boots the system BIOS, two Intel 8042s
(UPI-42, i.e. MCS-48 with the UPI register file) run the system and sound
controllers, and an 8749 scans the keyboard. All dumps below are the ones
MAME's `misc/mindset.cpp` expects, byte for byte.

## System BIOS (32 KB, two 16 KB EPROMs on a 16-bit bus)

| File | Role | Size | CRC32 |
|---|---|---|---|
| `mindset_u60_1.7_lo.bin` | system BIOS, even / low byte (EPROM U60) | 16,384 B | `00474dc1` |
| `mindset_u59_1.7_hi.bin` | system BIOS, odd / high byte (EPROM U59) | 16,384 B | `1434af10` |

- Reset entry at `0xFFFF0` (`0xF8000 + 0x7FF0`): `BA A0 FF B8 3C` = `mov dx,0xFFA0` / `mov ax,0xE03C` / `out dx,ax`
- Interleave direction: **U60 = even/low, U59 = odd/high**. Evidence: MAME loads
  `1.7_lo.u60` with `ROM_LOAD16_BYTE(..., 0, ...)` and `1.7_hi.u59` with
  `ROM_LOAD16_BYTE(..., 1, ...)`; with this pairing the image carries the strings
  `Mindset System Configuration` at `0xFEA92`, `(c) MINDSET Corp. 1983   Version 1.07`
  at `0xFA7E0` and `COPYRIGHT (C) MINDSET CORPORATION, 1983` at `0xFF600`; swapping
  the two gives garbage.
- Interleaved 32 KB image (synthesised, not stored): CRC32 `3d8445ce`, SHA256 `13780a05f34c4c1052f8cf13621796707b63df6c95389b7a149d0e4898d63b9f`, 8-bit sum `0x48`.
- The interleaved image is what the CPU sees at `0xF8000-0xFFFFF`; it is reproduced on
  demand with `flat[0::2] = u60; flat[1::2] = u59` and is deliberately not committed,
  so the repository keeps the raw EPROM dumps only.

## MCU firmware (2 KB each, MCS-48)

| File | Role | Size | CRC32 |
|---|---|---|---|
| `mindset_u17_253002.bin` | system 8042 firmware (253002-001, U17) | 2,048 B | `69da82c9` |
| `mindset_u16_253006.bin` | sound 8042 firmware (253006-001, U16) | 2,048 B | `7bea5edd` |
| `mindset_kbd_v3.0.bin` | 8749 keyboard firmware (v3.0) | 2,048 B | `1c6aa433` |

These are only needed when the two 8042s and the 8749 are run as real
processors (MAME does exactly that: its `i8042` device *is* the MCS-48 core).
They are not used by a high-level model of the mailbox.

## SHA256

```
c8c4d6ed250570ece6569f3794eae2c5f71611a2730f718039fd091d51ae4b2d  mindset_u60_1.7_lo.bin
6ac46f209b9a93c0afa2b18247539fc688fb2eae23168edd79e66c2e4128d9ad  mindset_u59_1.7_hi.bin
aff044f884bf5899284723516549e7ec8b460f06c930dca84f20e586e24052c4  mindset_u17_253002.bin
44800ab098a5295a64da4b349ca2e9a272983d414c0694e1f1e27294b9468c2f  mindset_u16_253006.bin
a64e3886ef1cb7c3341b932aa2b5483399052ff9c5fce7e5cf5d29fd3a5c9584  mindset_kbd_v3.0.bin
```

## Source

[bitsavers, Mindset firmware](https://bitsavers.org/pdf/mindset/firmware/) for the
system BIOS (`U59_1.7_HI.bin`, `U60_1.7_LO.bin`, `U16_253006-001.bin`,
`U17_253002-001.bin`) and [bitsavers, Mindset keyboard](https://bitsavers.org/pdf/mindset/keyboard/)
for `kbd_v3.0.bin`. Files are kept as downloaded and only renamed to record which
half of the bus they drive and which socket they came from.

## Usage (86Box)

```c
bios_load_interleaved("roms/machines/mindset/mindset_u60_1.7_lo.bin",
                      "roms/machines/mindset/mindset_u59_1.7_hi.bin",
                      0x000F8000, 0x8000, 0);
/* low/even file first: rom_load_interleaved(fnl, fnh, ...) */
/* the MCU images are loaded by their own device via rom_init(), not into the 80186 map */
```

The three MCU images belong to their own devices (`syscpu`, `soundcpu`, `kbdcpu`
regions in MAME) and have no address in the 80186 map.

