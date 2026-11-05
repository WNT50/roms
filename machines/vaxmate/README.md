# DEC VAXmate — system BIOS

The VAXmate keeps its BIOS in two 32 KB EPROMs sitting on a 16-bit bus: one device
supplies the low (even) bytes and the other the high (odd) bytes of every word.
Two revisions of the pair are archived here.

## 05/18/87 (later)

| File | Role | Size |
|---|---|---|
| `vaxmate_310E6_even.bin` | even / low byte (`310E6ROM`) | 32,768 B |
| `vaxmate_311E6_odd.bin` | odd / high byte (`311E6ROM`) | 32,768 B |

- Version: the image carries no version number. It identifies itself by the copyright
  string `Copyright Digital Equipment Corp. 1987` and the build date string
  `05/18/87` at `0xFFFF5`, so it is referred to as the **05/18/87 BIOS**.
- Reset vector at `0xFFFF0`: `EA 5B E0 00 F0` → `jmp F000:E05B`
- Interleave direction: **310E6 = even, 311E6 = odd**.

## 07/31/86 (earlier)

| File | Role | Size |
|---|---|---|
| `vaxmate_200E6_even.bin` | even / low byte (`23-200E6`) | 32,768 B |
| `vaxmate_199E6_odd.bin` | odd / high byte (`23-199E6`) | 32,768 B |

- Copyright string: `Copyright Digital Equipment Corp. 1986`, build date `07/31/86`
- Reset vector at `0xFFFF0`: `EA 5B E0 00 F0` → `jmp F000:E05B`
- Interleave direction: **200E6 = even, 199E6 = odd** — the opposite of the later
  pair, so the part numbers do not follow the even/odd order.

Pairing either set the wrong way round yields a different reset vector and no
copyright string at all, so the working combination is unambiguous in both cases.

SHA256:

```
5c4ab489a8fcbb6069096970cac8d18f531ab2b7d9e4401b31a5490571b0e922  vaxmate_310E6_even.bin
60cba614138155809226dd0fc30c1ca600e71c7b722e4cac291cff477845c458  vaxmate_311E6_odd.bin
6e0cd6c259dc627c9883c859d56462d9460088188cbc9b7a11738b132b12dd7f  vaxmate_200E6_even.bin
104214e4a34541c0ecd74e29dd0a610de55d4b0053f907a99b6e7c37679f7933  vaxmate_199E6_odd.bin
```

## Source

[bitsavers, DEC VAXmate firmware](https://bitsavers.org/pdf/dec/vaxmate/firmware/)
(`310E6ROM.BIN`, `311E6ROM.BIN`, `23-199e6.bin`, `23-200e6.bin`; directory dated
2020-01-05). The files are kept as downloaded and only renamed to record which half
of the bus they drive.

## Usage

86Box loads one pair with `bios_load_interleaved()` at `0x000F0000`, 64 KB
(`src/machine/m_vaxmate.c`); the machine currently ships the 05/18/87 pair, and the
07/31/86 pair is archived for reference.
