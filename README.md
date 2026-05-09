# Mario and Sonic at the Olympic Games series Modding Documentation

---

# Mario and Sonic at the Olympic Games (Wii)

## Overview

The game was co-developed by **Racjin**, known for the *Fullmetal Alchemist* PS2 trilogy and the *Naruto: Uzumaki Chronicles* series. As a result, its internal file structure closely mirrors that of other Racjin titles, and existing research on those games carries over well here.

I tried to summarize all the known info from all the places I could find.

---

## File Formats

The game's data is packed into `.DIG` archives at the disc root:

`CDDATA.DIG`, `CDDATA_##_##.DIG` (e.g. `CDDATA_EU_EN.DIG`)

### DIG

Container archive holding nearly all game data, except audio and video.

```
CDDATA.DIG
├── Header table (16 bytes per record, ends at record[0].offset × 0x800)
│   ├── offset         u32   sector index; × 0x800 = byte offset
│   ├── comp_size      u32   compressed size in bytes
│   ├── section_count  u16
│   ├── is_compressed  u16
│   └── decomp_size    u32   decompressed size in bytes
└── Data entries
    └── *.RAW
```

Compressed entries use the Racjin LZ algorithm (see [Credits](#credits)).

#### Extraction with [racjin-python](https://github.com/soyjxck/racjin-python)

```racjin extract --format cddata --big-endian CDDATA.DIG output/```

Note: racjin-python extracts .dig entries as .bin. I prefer to name them *.raw to match the Naruto modding scene.

### RAW

Secondary container holding a flat list of sections. Each section is either a single file, a pointer table of sub-files, a link, or empty.

```
*.RAW
├── Section headers (16 bytes each, count = first_section_offset / 16)
│   ├── type     u32
│   ├── size     u32
│   ├── offset   u32
│   └── padding  u32
└── Sections
    ├── [file]           single blob, identified by its content
    └── [pointer table]  directory of sub-files
        ├── count        u32
        ├── offset_N     u32
        └── size_N       u32
```

### File Types

> `*` marks names I assigned myself. Other names come from a file descriptor accidentally left in *Naruto: Uzumaki Chronicles*.

| Extension | Description |
|-----------|-------------|
| `.mb0` | Racjin binary tokenized text. Sequences of `u16` glyph indices and control codes, each terminated by `0x8000`. Prefixed by a `u32` sequence count and `u32` offsets into the word data. |
| `.utx`* | UTF-16 text strings. |
| `.img` | Racjin texture container. Wraps a `.tpl` image. |
| `.pap` | Racjin texture layout format. Defines coordinate layout and slicing of an atlas texture into sprites; may also contain sprite animations. Almost always paired with its `.img` — typically the entry immediately before it in the same section (`section[n][k]` -> `.pap`, `section[n][k+1]` -> `.img`). |


### File Types description

**`.img`**
```c
struct ImgHeader {
   u32 image_count;
   u32 tpl_offset;  // offset to embedded TPL (magic: 0x0020AF30)
};
```
Header is at `0x00` or `0x10` or even further. I have no idea why. TPL magic may also appear anywhere in the block.

**`.mb0`**
```c
struct Mb0Header {
   u32 sequence_count;
   u32 offsets[sequence_count];  // offsets[0] == 4 + sequence_count * 4
   u16 words[];                  // glyph indices and control codes, terminated by 0x8000
};
```

`0x8001` - \n, `0x8002` - iirc tabulation, other codes start from `0x81xx`. `0x900x` - Wii buttons icons index, think of a general glyph index for the buttons texture.


**`.pap`**
```c
struct PapHeader {
   u16 ca, cc, cb, cd;  // entry counts; at least one nonzero
   u32 oa, ob, oc, od;  // section offsets; first nonzero == 0x18
   // section entry strides: 0x0C, 0x04, 0x1C, 0x1C
};
```
TODO: Confirm the format. I derived this from my PS2 FMA notes.


**`.utx`**
```c
struct UtxHeader {
   u32 char_count;  // char_count * 2 == file size - 4. You can also understand it as UTF-16 characters count.
   u16 chars[];     // UTF-16, null-terminated
};
```

---

## Credits

### Racjin Format Research
*   [**Raw-man**](https://github.com/Raw-man)
*   [**SockNastre**](https://github.com/SockNastre)
*   [**Bit.Raiden**](https://github.com/MiguelQueiroz010)
*   [**NarutoClassics**](https://www.youtube.com/channel/UCcnKGBdUEAmr1ireju9yiCA) and everyone from their [*Discord*](https://discord.gg/Y2rFRJq)
*   [**JMDigital**](https://github.com/JenkinsTR)
*   [**Illidan**](https://github.com/Illidanz)

---

### Related Tools
*   [**SoyJack**](https://github.com/soyjxck))

---

# Mario and Sonic at the Olympic Winter Games (Wii)
Game uses NiGHTS: Journey Into Dreams formats
