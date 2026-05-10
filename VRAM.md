# VRAM Address and Allocation Best Practices

The SNES PPU (Picture Processing Unit) has **64 KiB of VRAM** (word addresses `$0000`–`$7FFF`). Every byte of it is shared between BG tile graphics, BG tilemaps, and OBJ (sprite) tile graphics. Nothing stops you from placing two things at the same VRAM (Video RAM) address — the hardware will silently corrupt both. 

## Hardware Constraints
### BG CHR base — BGxSC / BGxNBA registers

Background tile graphics must be placed on a **4 KiB boundary** (word address multiple of `$800`).

| Step | Word address |
|---|---|
| 1 | `$0800` |
| 2 | `$1000` |
| 3 | `$1800` |
| … | … |
| 16 | `$7800` |

PVSneslib exposes function `bgSetGfxPtr` to change VRAM address of backgrounds.

### BG tilemap base — BGxSC register

Background tilemaps must be placed on a **2 KiB boundary** (word address multiple of `$400`). Each 32×32 tilemap occupies exactly 2 KiB (`$400` words). Larger maps (32×64, 64×32, 64×64) occupy 2, 3, or 4 contiguous 2 KiB blocks.

| Map size | Words | Word address span |
|---|---|---|
| 32×32 | `$400` | 1 block |
| 64×32 or 32×64 | `$800` | 2 blocks |
| 64×64 | `$1000` | 4 blocks |

PVSneslib exposes  function `bgSetMapPtr` to change VRAM address of backgrounds.

### OBJ (Sprite) CHR base — OBSEL `$2101`

Sprite tile data must start on an **8 KiB boundary** (word address multiple of `$1000`). The OBSEL register encodes the base in 3 bits (bits 2–0), each step representing 4 KiW = 8 KiB:

| OBSEL bits 2–0 | Word address | Notes |
|---|---|---|
| `000` | `$0000` | PVSnesLib default for large sprites |
| `001` | `$1000` | PVSnesLib default for small sprites |
| `010` | `$2000` | |
| `011` | `$3000` | |
| `100` | `$4000` | |
| `101` | `$5000` | |
| `110` | `$6000` | |
| `111` | `$7000` | |

Passing a non-aligned value to `oamInitGfxSet()` or `oamInitGfxAttr()` causes OBSEL to round silently — your sprites will read tiles from the wrong location.

OBSEL bits 4–3 select the **second OBJ name table** offset, in units of 4 KiW relative to the base. This matters when your sprites need more than 256 distinct tiles on screen at once. PVSnesLib sets these bits as part of the `oamsize` parameter.

### Tile Size in VRAM

Knowing how much space each asset occupies lets you pack the map accurately:

| Format | Bytes per 8×8 tile |
|---|---|
| 2bpp (4 colors) | 16 |
| 4bpp (16 colors) | 32 |
| 8bpp (256 colors) | 64 |

OBJ tiles are always 4bpp in Modes 1–7. BG tile depth depends on the active background mode.

## Example VRAM Layout (Mode 1)

The layout below avoids all overlaps for a typical Mode 1 game using two scrolling BGs, a text/HUD layer, and sprites. Addresses are VRAM word addresses.

```
$0000–$0FFF   OBJ CHR (sprites, large size 4bpp)         8 KiW = 16 KiB = 512 tiles
              oamInitGfxSet(..., 0x0000, ...)

$1000–$17FF   OBJ CHR (sprites, small size 4bpp)         4 KiW = 4 KiB = 256 tiles
              oamInitGfxSet(..., 0x0000, ...)

$1800–$1FFF   (free)

$2000–$27FF   BG1 CHR (gameplay, 4bpp)         4 KiW = 8 KiB = 256 tiles
              bgSetGfxPtr(0, 0x2000)

$2800–$2FFF   BG2 CHR (gameplay, 4bpp)         4 KiW = 8 KiB = 256 tiles
              bgSetGfxPtr(1, 0x2800)

$3000–$33FF   Console font CHR (2bpp, ~96 tiles) 2 KiW
              consoleSetTextGfxPtr(0x3000)

$3400–$5FFF   (free — extra CHR, animation frames)

$6000–$63FF   BG3 tilemap 32×32               1 KiW = 2 KiB
              bgSetMapPtr(2, 0x6000, SC_32x32)

$6400–$67FF   BG2 tilemap 32×32               1 KiW = 2 KiB
              bgSetMapPtr(1, 0x6400, SC_32x32)

$6800–$6BFF   Console text tilemap 32×32      1 KiW = 2 KiB
              consoleSetTextMapPtr(0x6800)

$6C00–$6FFF   (free)

$7000–$73FF   BG1 tilemap 32×32               1 KiW = 2 KiB
              bgSetMapPtr(0, 0x7000, SC_32x32)

$7400–$7FFF   (free — extend BG1 map to 64×32 here if needed)
```
