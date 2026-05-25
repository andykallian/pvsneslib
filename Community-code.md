This page has been done to group code we saw everywhere. It contains some complete projects or sometimes only sample code made by the community but all can can be interesting to study.

Please keep in mind that some of them are not maintained, it can have a gap with the latest version of PVsneslib and do not build correctly.

Feel free to contribute and add a link to your project !

### Sound test project :

[This code](https://github.com/alekmaul/pvsneslib/files/6565523/SoundTest_ByDiegoLeao_PvsneslibV3.zip) made by diegoleao with PVSneslib v3 will explain you how to manage the audio part with the library (the "Readme.txt" file also have useful information that is not present in the code itself). 
If you need the previous version of SoundTest (for PVSneslib v2) you can find it [here](https://github.com/alekmaul/pvsneslib/files/5396843/SoundTest_ByDiegoLeao.zip).


### Street of rage demo port

Odelot shared his port of SoR2 which is now cancelled, it has been made with pvsneslib v2 : https://github.com/odelot/sor2_snes

### Bomberworld

1r3n33 created a game which is available on his github: https://github.com/1r3n33/bomberworld

It is very useful to see how you can organize your code!

### The last super

Dr. Ludos created it for a game jam then continued to improve it, the code is well commented. The original project is available on his website but it was build for PVSneslib v2: https://drludos.itch.io/the-last-super

It has been ported on PVSneslib v3 and is available [here](https://github.com/alekmaul/pvsneslib/files/6556286/TheLastSuper_sourcecode.zip)

### Scrolling map with collisions :

[This code](https://github.com/alekmaul/pvsneslib/files/7149024/mapscroll_example_with_collisions_v3.zip) made by diegoleao changes the original "map scroll example" to add collision checking. Move Mario up, down, left, and right to check for tiles that contain collisions. Press and hold B to move 1 pixel at a time, for finer control.

### Update the priority of a tile map area

#### Description

This code made by malayli updates a tile map area priority in any backgrounds like BG1 or BG2.
It's very useful if you want to have tiles appearing over and behind your sprites for example.

#### Code

```
u16 bgMapTemp[4096];
u16 bgMapPageNumberSize;
u8 bgMapX;
u16 bgMapY;
u16 bgMapYMax;

/*!\brief Set the priority of a tile map area at {x, y, width, height, pageNumber}
    \param tileMap the tile map
    \param priority the priority to be set for the tile map area
    \param x the X position of the tile map area
    \param y the Y position of the tile map area
    \param width the width of the tile map area
    \param height the height of the tile map area
    \param pageNumber the page number of the tile map
    \return the updated tile map
*/
char * updateTileMapAreaWithPriority(char *tileMap, u8 priority, u16 x, u16 y, u16 width, u16 height, u8 pageNumber) {
    bgMapPageNumberSize = 1024 * pageNumber;
    bgMapX = 0;
    bgMapY = (y * 32) + bgMapPageNumberSize;
    bgMapYMax = (y * 32) + (height * 32) + bgMapPageNumberSize;

    memcpy((u8 *) &bgMapTemp, tileMap, 4096);

    while(bgMapY < bgMapYMax) {
        if (x <= bgMapX && bgMapX < width) {
            bgMapTemp[bgMapY] = ((u16 *)tileMap)[bgMapY] | (priority<<13);
        }

        if (bgMapX == 31) {
            bgMapX = 0;

        } else {
            bgMapX++;
        }

        bgMapY++;
    }

    return (char *)bgMapTemp;
}
```

#### Example

![Update the priority of a tile map area](https://user-images.githubusercontent.com/48180545/187201833-17357bb2-eed4-48b8-a3b1-64a6538ece08.png)



# Large 4096x4096 Mode 7 World Streaming Demo

**Author:** [Anderson Viana](https://github.com/andykallian)

---

## Description

This example demonstrates a method for bypassing the SNES Mode 7 1024×1024 map limitation by dividing a larger world into multiple chunks. The world size used here is 4096×4096 and is divided into 16 chunks arranged in a 4×4 grid.

Using a unique tileset and palette for each chunk would require expensive VRAM updates and could introduce stuttering when crossing chunk boundaries. To avoid this, all chunks share the same tileset and palette resources. VRAM updates dynamically as the player moves, allowing seamless transitions between chunks.

Two Python scripts were created to automate the asset pipeline. The first script, `convert_Overworld`, processes a PNG image and generates:

- `palette.bin`
- `tiles.bin`
- `map.bin`

A second script takes `map.bin` and splits it into 16 chunk files.

---

## Mode 7 Limitation

`convert_Overworld` only works if the source "bigMap" uses a maximum of 256 unique 8×8 tiles. If the map contains 257 or more unique tiles, file generation is aborted. This restriction was intentionally implemented because Mode 7 itself is limited to 256 tiles.

---

## Python Scripts

### `convert_overworld.py`

Processes a 4096×4096 PNG overworld image and outputs `ow_tiles.bin`, `ow_map.bin`, and `ow_palette.bin` in SNES-compatible formats.

```python
from PIL import Image
import struct
import os
import time

IMG_PATH = "your map.png address here" #folder1/folder2/overworld.png 
OUT_TILES = "the address where you want to save the ow_tiles.bin file" #folder1/folder2/overworld_tiles.bin
OUT_MAP   = "the address where you want to save the ow_map.bin file" #folder1/folder2/overworld_map.bin
OUT_PAL   = "the address where you want to save the ow_palette.bin file" #folder1/folder2/overworld_palette.bin

TILE_SIZE = 8
MAP_W = 512
MAP_H = 512

print("=" * 50)
print("  SNES Mode 7 Overworld Converter")
print("=" * 50)

# =========================================
# CARREGAR IMAGEM
# =========================================
print(f"\n[1/5] Carregando imagem: {IMG_PATH}")
t = time.time()
img = Image.open(IMG_PATH).convert("RGB")
print(f"      Resolucao: {img.width}x{img.height} pixels")
print(f"      Tempo: {time.time()-t:.2f}s")
assert img.width == 4096 and img.height == 4096, "Imagem deve ser 4096x4096"

# =========================================
# PALETA
# =========================================
print(f"\n[2/5] Extraindo paleta...")
t = time.time()
color_set = set()
for y in range(img.height):
    for x in range(img.width):
        color_set.add(img.getpixel((x, y)))

print(f"      Cores originais encontradas: {len(color_set)}")

if len(color_set) > 255:
    print(f"      AVISO: mais de 255 cores, reduzindo com quantize...")
    img_p = img.quantize(colors=255, method=Image.Quantize.MEDIANCUT)
    img = img_p.convert("RGB")
    color_set = set()
    for y in range(img.height):
        for x in range(img.width):
            color_set.add(img.getpixel((x, y)))
    print(f"      Cores apos quantize: {len(color_set)}")

colors = list(color_set)
palette = [(0, 0, 0)] + [c for c in colors if c != (0, 0, 0)]
palette = palette[:256]
color_to_idx = {c: i for i, c in enumerate(palette)}
print(f"      Paleta final: {len(palette)} cores")
print(f"      Cor 0 (backdrop): {palette[0]}")
print(f"      Cor 1: {palette[1]}")
print(f"      Cor 255: {palette[-1]}")
print(f"      Tempo: {time.time()-t:.2f}s")

# =========================================
# TILES
# =========================================
print(f"\n[3/5] Extraindo tiles ({MAP_W}x{MAP_H} = {MAP_W*MAP_H} total)...")
t = time.time()
tile_map = {}
tiles_list = []
map_data = []
overflow_count = 0

for ty in range(MAP_H):
    if ty % 64 == 0:
        print(f"      Processando linha {ty}/{MAP_H}...")
    for tx in range(MAP_W):
        px = tx * TILE_SIZE
        py = ty * TILE_SIZE
        tile_pixels = []
        for y in range(TILE_SIZE):
            for x in range(TILE_SIZE):
                rgb = img.getpixel((px + x, py + y))
                idx = color_to_idx.get(rgb, 0)
                tile_pixels.append(idx)
        tile_bytes = bytes(tile_pixels)
        if tile_bytes not in tile_map:
            tile_map[tile_bytes] = len(tiles_list)
            tiles_list.append(tile_bytes)
        tidx = tile_map[tile_bytes]
        if tidx > 255:
            overflow_count += 1
        map_data.append(tidx)

print(f"      Tiles unicos encontrados: {len(tiles_list)}")
print(f"      Tiles que cabem na VRAM (<=256): {min(len(tiles_list), 256)}")
if overflow_count > 0:
    print(f"      AVISO: {overflow_count} referencias de mapa apontam para tiles >255 (serao truncadas)")
else:
    print(f"      OK: todos os tiles cabem em 256 indices")
print(f"      Tempo: {time.time()-t:.2f}s")

# =========================================
# SALVAR TILES
# =========================================
print(f"\n[4/5] Salvando arquivos...")
t = time.time()

tiles_bin = bytearray()
for tile in tiles_list[:256]:
    tiles_bin += tile
while len(tiles_bin) < 256 * 64:
    tiles_bin += bytes(64)

with open(OUT_TILES, "wb") as f:
    f.write(tiles_bin)
print(f"      ow_tiles.bin: {len(tiles_bin)} bytes ({len(tiles_bin)//64} tiles)")

with open(OUT_MAP, "wb") as f:
    for idx in map_data:
        f.write(bytes([idx % 256]))
print(f"      ow_map.bin: {len(map_data)} bytes ({MAP_W}x{MAP_H} tiles)")

pal_bin = bytearray()
for r, g, b in palette:
    r5 = r >> 3
    g5 = g >> 3
    b5 = b >> 3
    color = (b5 << 10) | (g5 << 5) | r5
    pal_bin += struct.pack("<H", color)

with open(OUT_PAL, "wb") as f:
    f.write(pal_bin)
print(f"      ow_palette.bin: {len(pal_bin)} bytes ({len(pal_bin)//2} cores)")
print(f"      Tempo: {time.time()-t:.2f}s")

# =========================================
# RESUMO
# =========================================
print(f"\n[5/5] Resumo final:")
print(f"      Mapa total:     {MAP_W*TILE_SIZE}x{MAP_H*TILE_SIZE} pixels")
print(f"      Tiles unicos:   {len(tiles_list)}")
print(f"      Cores na paleta:{len(palette)}")
print(f"      Chunks 128x128: {(MAP_W//128)*(MAP_H//128)} chunks ({MAP_W//128}x{MAP_H//128} grid)")
print(f"      VRAM por chunk: 16384 bytes mapa + 16384 bytes tiles = 32KB")
if len(tiles_list) > 256:
    print(f"\n      !! ATENCAO: {len(tiles_list)} tiles unicos encontrados.")
    print(f"         O SNES Mode 7 suporta apenas 256.")
    print(f"         Sera necessario streaming de tileset por chunk.")
else:
    print(f"\n      Tileset cabe inteiro na VRAM.")
print("\n" + "=" * 50)
print("  Conversao concluida!")
print("=" * 50)
```

---

### `split_chunks.py`

Takes `ow_map.bin` and splits it into 16 chunk files arranged in a 4×4 grid.

```python
MAP_W = 512      # largura total em tiles
MAP_H = 512      # altura total em tiles
CHUNK_SIZE = 128 # tiles por chunk

with open("your map.bin adress here", "rb") as f: #folder1/folder2/overworld_map.bin
    data = f.read()

# converte para grid 2D
map_grid = []
for y in range(MAP_H):
    row = []
    for x in range(MAP_W):
        row.append(data[y * MAP_W + x])
    map_grid.append(row)

# extrai chunks respeitando a grade 2D
for chunk_row in range(4):
    for chunk_col in range(4):
        chunk = bytearray()
        for y in range(CHUNK_SIZE):
            for x in range(CHUNK_SIZE):
                tile_x = chunk_col * CHUNK_SIZE + x
                tile_y = chunk_row * CHUNK_SIZE + y
                chunk.append(map_grid[tile_y][tile_x])
        filename = f"../maps/ow_chunk_{chunk_row}_{chunk_col}.bin"
        with open(filename, "wb") as f:
            f.write(chunk)
        print(f"chunk_{chunk_row}_{chunk_col}.bin gerado ({len(chunk)} bytes)")

print("Pronto!")
```

---

## `overworld.c`

This file contains the world chunk management and streaming systems. Chunk loading occurs dynamically and updates VRAM without visible stuttering when crossing chunk boundaries. VRAM updates are tied directly to player movement.

Additional systems include tile scrolling features used for animation effects such as:

- water
- waterfalls
- partial tile scrolling

Partial scrolling allows animation to affect only half of a tile horizontally or vertically.

```c

#include <snes.h>
#include "overworld.h"

//=========================================================
// DEFINES
//=========================================================

#define WORLD_SIZE   512    // Total map size in tiles (512x512)
#define VIEW_SIZE    128    // Visible VRAM window size in tiles
#define TILE_SIZE      8    // Tile size in pixels
#define UPDATE_ROW     1    // Queue entry type: horizontal row update
#define UPDATE_COL     2    // Queue entry type: vertical column update
#define MAX_UPDATES    8    // Maximum tile updates allowed per frame

//=========================================================
// HARDWARE REGISTER ALIASES
//=========================================================

#define REG_VMADDL  (*(volatile u8*)0x2116)  // VRAM address low byte
#define REG_VMADDH  (*(volatile u8*)0x2117)  // VRAM address high byte
#define REG_DMAP0   (*(volatile u8*)0x4300)  // DMA channel 0 parameters
#define REG_BBAD0   (*(volatile u8*)0x4301)  // DMA channel 0 B-Bus destination
#define REG_A1T0L   (*(volatile u8*)0x4302)  // DMA channel 0 source address low
#define REG_A1T0H   (*(volatile u8*)0x4303)  // DMA channel 0 source address high
#define REG_A1B0    (*(volatile u8*)0x4304)  // DMA channel 0 source bank
#define REG_DAS0L   (*(volatile u8*)0x4305)  // DMA channel 0 byte count low
#define REG_DAS0H   (*(volatile u8*)0x4306)  // DMA channel 0 byte count high
#define REG_MDMAEN  (*(volatile u8*)0x420B)  // CPU DMA enable register

//=========================================================
// EXTERNAL ROM DATA
//=========================================================

extern char ow_tiles_bin, ow_tiles_bin_end;  // Overworld tileset graphics
extern char ow_palette_bin;                  // Overworld color palette
extern char ow_map_chunk0,  ow_map_chunk1,  ow_map_chunk2,  ow_map_chunk3;
extern char ow_map_chunk4,  ow_map_chunk5,  ow_map_chunk6,  ow_map_chunk7;
extern char ow_map_chunk8,  ow_map_chunk9,  ow_map_chunk10, ow_map_chunk11;
extern char ow_map_chunk12, ow_map_chunk13, ow_map_chunk14, ow_map_chunk15;

//=========================================================
// MAP CHUNK TABLE
//=========================================================

// 4x4 grid of 128x128-tile chunks covering the full 512x512 world
static u8* chunk_table[4][4] =
{
    { (u8*)&ow_map_chunk0,  (u8*)&ow_map_chunk1,  (u8*)&ow_map_chunk2,  (u8*)&ow_map_chunk3  },
    { (u8*)&ow_map_chunk4,  (u8*)&ow_map_chunk5,  (u8*)&ow_map_chunk6,  (u8*)&ow_map_chunk7  },
    { (u8*)&ow_map_chunk8,  (u8*)&ow_map_chunk9,  (u8*)&ow_map_chunk10, (u8*)&ow_map_chunk11 },
    { (u8*)&ow_map_chunk12, (u8*)&ow_map_chunk13, (u8*)&ow_map_chunk14, (u8*)&ow_map_chunk15 }
};

//=========================================================
// DATA STRUCTURES
//=========================================================

// Pending VRAM tile row or column write queued for VBlank
typedef struct
{
    u8  type;    // UPDATE_ROW or UPDATE_COL
    u8  addr;    // Target row or column index in the VRAM window
    u8* buffer;  // Pointer to the tile data to upload
} MapUpdate;

//=========================================================
// STATE
//=========================================================

static s16 world_x = 0;          // Top-left X of the VRAM window in tile coordinates
static s16 world_y = 0;          // Top-left Y of the VRAM window in tile coordinates
static u8  vram_left = 0;        // Current horizontal scroll offset within the VRAM ring buffer
static u8  vram_top  = 0;        // Current vertical scroll offset within the VRAM ring buffer
static u16 world_zoom = 0x0100;  // Saved Mode7 zoom factor for restoring after airship mode

//=========================================================
// DOUBLE BUFFERS
//=========================================================

static u8 row_buf[2][128];  // Ping-pong buffers for building incoming tile rows
static u8 col_buf[2][128];  // Ping-pong buffers for building incoming tile columns
static u8 row_flip = 0;     // Selects which row buffer is currently being written
static u8 col_flip = 0;     // Selects which column buffer is currently being written

//=========================================================
// UPDATE QUEUE
//=========================================================

static MapUpdate updates[MAX_UPDATES];  // Pending VBlank VRAM writes for this frame
static u8 update_count = 0;             // Number of pending updates in the queue

//=========================================================
// WATER ANIMATION
//=========================================================

static u8 waterbuf[832];  // RAM copy of all animated water tile pixel data
u8 water_timer = 0;       // Frame counter; animation advances every 8 frames

//=========================================================
// DMA HELPERS
//=========================================================

// Transfers a block of WRAM data to VRAM using CPU DMA channel 0
static inline void dmaCopyVramFast(u16 vram_addr, u8* src, u16 size, u8 mode)
{
    *(volatile u8*)0x2115 = mode;  // Set VRAM increment mode

    REG_VMADDL = vram_addr & 0xFF;       // Set VRAM destination address low
    REG_VMADDH = vram_addr >> 8;         // Set VRAM destination address high

    REG_DMAP0 = 0x00;                    // Single byte transfer mode
    REG_BBAD0 = 0x18;                    // Destination: VMDATAL ($2118)

    REG_A1T0L = (u16)src & 0xFF;         // Source address low
    REG_A1T0H = ((u16)src >> 8) & 0xFF;  // Source address high
    REG_A1B0  = 0x7E;                    // Source bank: WRAM $7E

    REG_DAS0L = size & 0xFF;  // Transfer byte count low
    REG_DAS0H = size >> 8;    // Transfer byte count high

    REG_MDMAEN = 0x01;  // Trigger DMA channel 0
}

//=========================================================
// TILE BUILDERS
//=========================================================

// Fills dst with 128 tile IDs along a horizontal row at world position (wx, wy)
static void buildRowFast(u8* dst, u16 wx, u16 wy)
{
    u16 remaining = 128;
    u16 out = 0;

    while (remaining)
    {
        u16 chunk_x = wx >> 7;   // Which chunk column (0-3)
        u16 chunk_y = wy >> 7;   // Which chunk row (0-3)

        u8* chunk  = chunk_table[chunk_y][chunk_x];
        u16 localx = wx & 127;
        u16 localy = wy & 127;
        u16 run    = 128 - localx;  // Tiles remaining in this chunk before wrapping

        if (run > remaining)
            run = remaining;

        u16 j;
        u16 src_index = (localy << 7) | localx;
        for (j = 0; j < run; j++)
            dst[out + j] = chunk[src_index + j];

        wx = (wx + run) & 511;  // Wrap X to world boundary
        out       += run;
        remaining -= run;
    }
}

// Fills dst with 128 tile IDs along a vertical column at world position (wx, wy)
static void buildColumnFast(u8* dst, u16 wx, u16 wy)
{
    u16 i;
    u16 localx  = wx & 127;
    u16 chunk_x = wx >> 7;

    for (i = 0; i < 128; i++)
    {
        u16 y       = (wy + i) & 511;
        u16 chunk_y = y >> 7;
        u8* chunk   = chunk_table[chunk_y][chunk_x];
        u16 localy  = y & 127;
        dst[i]      = chunk[(localy << 7) | localx];
    }
}

// Adds a VRAM write command to the per-frame update queue
static inline void queueUpdate(u8 type, u8 addr, u8* buf)
{
    if (update_count >= MAX_UPDATES)
        return;

    updates[update_count].type   = type;
    updates[update_count].addr   = addr;
    updates[update_count].buffer = buf;
    update_count++;
}

// Uploads the entire 128x128 VRAM window from current world position
static void uploadFull(void)
{
    u16 y;
    for (y = 0; y < 128; y++)
    {
        buildRowFast(row_buf[0], world_x, (world_y + y) & 511);
        dmaCopyVramFast(y << 7, row_buf[0], 128, 0x00);
    }
}

//=========================================================
// WATER ANIMATION
//=========================================================

// Shifts pixel rows/columns in waterbuf to simulate flowing water
void overworldAnimateWaterTick(void)
{
    water_timer++;
    if (water_timer < 8)
        return;
    water_timer = 0;

    u16 tile, row, col;
    u8 temp;

    // Tiles 0-3: full water tile, rotate pixels left horizontally
    for (tile = 0; tile < 4; tile++)
    {
        u16 base = tile << 6;
        for (row = 0; row < 8; row++)
        {
            u16 p = base + (row << 3);
            temp = waterbuf[p+7];
            waterbuf[p+7]=waterbuf[p+6]; waterbuf[p+6]=waterbuf[p+5];
            waterbuf[p+5]=waterbuf[p+4]; waterbuf[p+4]=waterbuf[p+3];
            waterbuf[p+3]=waterbuf[p+2]; waterbuf[p+2]=waterbuf[p+1];
            waterbuf[p+1]=waterbuf[p+0]; waterbuf[p]=temp;
        }
    }

    // Tiles 0x98-0x99: top half only, rotate pixels left horizontally
    for (tile = 4; tile < 6; tile++)
    {
        u16 base = tile << 6;
        for (row = 0; row < 4; row++)
        {
            u16 p = base + (row << 3);
            temp = waterbuf[p+7];
            waterbuf[p+7]=waterbuf[p+6]; waterbuf[p+6]=waterbuf[p+5];
            waterbuf[p+5]=waterbuf[p+4]; waterbuf[p+4]=waterbuf[p+3];
            waterbuf[p+3]=waterbuf[p+2]; waterbuf[p+2]=waterbuf[p+1];
            waterbuf[p+1]=waterbuf[p+0]; waterbuf[p]=temp;
        }
    }

    // Tiles 0x90-0x91: bottom half only, rotate pixels left horizontally
    for (tile = 6; tile < 8; tile++)
    {
        u16 base = tile << 6;
        for (row = 4; row < 8; row++)
        {
            u16 p = base + (row << 3);
            temp = waterbuf[p+7];
            waterbuf[p+7]=waterbuf[p+6]; waterbuf[p+6]=waterbuf[p+5];
            waterbuf[p+5]=waterbuf[p+4]; waterbuf[p+4]=waterbuf[p+3];
            waterbuf[p+3]=waterbuf[p+2]; waterbuf[p+2]=waterbuf[p+1];
            waterbuf[p+1]=waterbuf[p+0]; waterbuf[p]=temp;
        }
    }

    // Tiles 0xDB-0xDE: full tiles, rotate pixels upward vertically
    for (tile = 8; tile < 12; tile++)
    {
        u16 base = tile << 6;
        for (col = 0; col < 8; col++)
        {
            temp = waterbuf[base+(7<<3)+col];
            waterbuf[base+(7<<3)+col]=waterbuf[base+(6<<3)+col];
            waterbuf[base+(6<<3)+col]=waterbuf[base+(5<<3)+col];
            waterbuf[base+(5<<3)+col]=waterbuf[base+(4<<3)+col];
            waterbuf[base+(4<<3)+col]=waterbuf[base+(3<<3)+col];
            waterbuf[base+(3<<3)+col]=waterbuf[base+(2<<3)+col];
            waterbuf[base+(2<<3)+col]=waterbuf[base+(1<<3)+col];
            waterbuf[base+(1<<3)+col]=waterbuf[base+(0<<3)+col];
            waterbuf[base+(0<<3)+col]=temp;
        }
    }

    // Tile 0xD6: single tile, rotate pixels upward vertically
    u16 base_D6 = 12 << 6;
    for (col = 0; col < 8; col++)
    {
        temp = waterbuf[base_D6+(7<<3)+col];
        waterbuf[base_D6+(7<<3)+col]=waterbuf[base_D6+(6<<3)+col];
        waterbuf[base_D6+(6<<3)+col]=waterbuf[base_D6+(5<<3)+col];
        waterbuf[base_D6+(5<<3)+col]=waterbuf[base_D6+(4<<3)+col];
        waterbuf[base_D6+(4<<3)+col]=waterbuf[base_D6+(3<<3)+col];
        waterbuf[base_D6+(3<<3)+col]=waterbuf[base_D6+(2<<3)+col];
        waterbuf[base_D6+(2<<3)+col]=waterbuf[base_D6+(1<<3)+col];
        waterbuf[base_D6+(1<<3)+col]=waterbuf[base_D6+(0<<3)+col];
        waterbuf[base_D6+(0<<3)+col]=temp;
    }
}

// DMAs all animated water tile data from waterbuf to their VRAM locations
void overworldAnimateWaterFlush(void)
{
    *(volatile u8*)0x2115 = 0x80;  // VRAM increment by word after high byte write
    REG_DMAP0 = 0x00;
    REG_BBAD0 = 0x19;  // Destination: VMDATAH ($2119)
    REG_A1B0  = 0x7E;  // Source bank: WRAM

    // Tiles 0-3 -> VRAM 0x0000
    REG_VMADDL=0x00; REG_VMADDH=0x00;
    REG_A1T0L=(u16)&waterbuf[0]&255; REG_A1T0H=((u16)&waterbuf[0]>>8)&255;
    REG_DAS0L=256&0xFF; REG_DAS0H=256>>8; REG_MDMAEN=1;

    // Tiles 0x98-0x99 -> VRAM 0x25C0
    REG_VMADDL=0xC0; REG_VMADDH=0x25;
    REG_A1T0L=(u16)&waterbuf[256]&255; REG_A1T0H=((u16)&waterbuf[256]>>8)&255;
    REG_DAS0L=128&0xFF; REG_DAS0H=128>>8; REG_MDMAEN=1;

    // Tiles 0x90-0x91 -> VRAM 0x2400
    REG_VMADDL=0x00; REG_VMADDH=0x24;
    REG_A1T0L=(u16)&waterbuf[384]&255; REG_A1T0H=((u16)&waterbuf[384]>>8)&255;
    REG_DAS0L=128&0xFF; REG_DAS0H=128>>8; REG_MDMAEN=1;

    // Tiles 0xDB-0xDE -> VRAM 0x36C0
    REG_VMADDL=0xC0; REG_VMADDH=0x36;
    REG_A1T0L=(u16)&waterbuf[512]&255; REG_A1T0H=((u16)&waterbuf[512]>>8)&255;
    REG_DAS0L=256&0xFF; REG_DAS0H=256>>8; REG_MDMAEN=1;

    // Tile 0xD6 -> VRAM 0x3580
    REG_VMADDL=0x80; REG_VMADDH=0x35;
    REG_A1T0L=(u16)&waterbuf[768]&255; REG_A1T0H=((u16)&waterbuf[768]>>8)&255;
    REG_DAS0L=64&0xFF; REG_DAS0H=64>>8; REG_MDMAEN=1;
}

//=========================================================
// SYSTEM CONTROL
//=========================================================

// Initializes overworld: loads assets, builds VRAM, copies water tiles to RAM
void overworldInit(s16 cam_x, s16 cam_y, u16 zoom)
{
    // Load tileset and initial map chunk into Mode7 VRAM layout
    bgInitMapTileSet7(&ow_tiles_bin, &ow_map_chunk0, &ow_palette_bin,
        (&ow_tiles_bin_end - &ow_tiles_bin), 0x0000);

    setMode7(0);
    REG_M7SEL = M7_WRAP;  // Wrap Mode7 plane at map edges

    // Initialize VRAM ring-buffer origin from camera position
    world_x   = ((cam_x >> 3) - 64) & 511;
    world_y   = ((cam_y >> 3) - 64) & 511;
    vram_left = 0;
    vram_top  = 0;

    uploadFull();  // Fill entire VRAM window with current world tiles

    // Copy animated water tile pixel data from ROM into WRAM for manipulation
    char* rom_src = (char*)&ow_tiles_bin;
    memcpy(&waterbuf[0],   rom_src,           256);   // Tiles 0-3
    memcpy(&waterbuf[256], rom_src + 9728,    128);   // Tiles 0x98-0x99
    memcpy(&waterbuf[384], rom_src + 9216,    128);   // Tiles 0x90-0x91
    memcpy(&waterbuf[512], rom_src + 14016,   256);   // Tiles 0xDB-0xDE
    memcpy(&waterbuf[768], rom_src + 13696,    64);   // Tile 0xD6

    // Save zoom and apply it to Mode7 matrix
    world_zoom = zoom;
    REG_M7A = zoom & 0xFF; REG_M7A = zoom >> 8;
    REG_M7D = zoom & 0xFF; REG_M7D = zoom >> 8;
}

// Restores the Mode7 scale matrix to the saved overworld zoom factor
void overworldRestoreZoom(void)
{
    REG_M7A = world_zoom & 0xFF; REG_M7A = world_zoom >> 8;
    REG_M7D = world_zoom & 0xFF; REG_M7D = world_zoom >> 8;
}

// Updates BG scroll registers to match current camera position
void overworldScroll(s16 cam_x, s16 cam_y)
{
    s16 lx = (vram_left << 3) + (cam_x - (world_x << 3));
    s16 ly = (vram_top  << 3) + (cam_y - (world_y << 3));

    REG_M7HOFS = lx & 0xFF; REG_M7HOFS = lx >> 8;
    REG_M7VOFS = ly & 0xFF; REG_M7VOFS = ly >> 8;
}

// Returns the current scroll values without writing to hardware registers
void overworldGetScroll(s16 cam_x, s16 cam_y, s16 *sx, s16 *sy)
{
    *sx = (vram_left << 3) + (cam_x - (world_x << 3));
    *sy = (vram_top  << 3) + (cam_y - (world_y << 3));
}

// Detects camera movement and enqueues tile row/column updates needed for scroll
void overworldPrepare(s16 cam_x, s16 cam_y)
{
    s16 target_x = ((cam_x >> 3) - 64) & 511;
    s16 target_y = ((cam_y >> 3) - 64) & 511;

    s16 dx = (target_x - world_x) & 511;
    s16 dy = (target_y - world_y) & 511;

    if (dx > 255) dx -= 512;  // Convert to signed delta
    if (dy > 255) dy -= 512;

    update_count = 0;

    // Scroll right: advance world_x and build new right-edge column
    while (dx > 0)
    {
        u8* buf = col_buf[col_flip];
        world_x = (world_x + 1) & 511;
        u8 write_col = vram_left;
        vram_left = (vram_left + 1) & 127;
        buildColumnFast(buf, (world_x + 127) & 511, world_y);
        queueUpdate(UPDATE_COL, write_col, buf);
        col_flip ^= 1;
        dx--;
    }

    // Scroll left: retreat world_x and build new left-edge column
    while (dx < 0)
    {
        u8* buf = col_buf[col_flip];
        world_x   = (world_x - 1) & 511;
        vram_left = (vram_left - 1) & 127;
        buildColumnFast(buf, world_x, world_y);
        queueUpdate(UPDATE_COL, vram_left, buf);
        col_flip ^= 1;
        dx++;
    }

    // Scroll down: advance world_y and build new bottom-edge row
    while (dy > 0)
    {
        u8* buf = row_buf[row_flip];
        world_y = (world_y + 1) & 511;
        u8 write_row = vram_top;
        vram_top = (vram_top + 1) & 127;
        buildRowFast(buf, world_x, (world_y + 127) & 511);
        queueUpdate(UPDATE_ROW, write_row, buf);
        row_flip ^= 1;
        dy--;
    }

    // Scroll up: retreat world_y and build new top-edge row
    while (dy < 0)
    {
        u8* buf = row_buf[row_flip];
        world_y  = (world_y - 1) & 511;
        vram_top = (vram_top - 1) & 127;
        buildRowFast(buf, world_x, world_y);
        queueUpdate(UPDATE_ROW, vram_top, buf);
        row_flip ^= 1;
        dy++;
    }
}

// Executes all queued VRAM row/column writes (called inside VBlank)
void overworldFlush(void)
{
    u8 i;
    for (i = 0; i < update_count; i++)
    {
        MapUpdate* u = &updates[i];

        if (u->type == UPDATE_ROW)
        {
            // Split row upload around VRAM ring-buffer wrap point
            u8 part1 = 128 - vram_left;
            u8 part2 = vram_left;
            dmaCopyVramFast(((u16)u->addr << 7) + vram_left, u->buffer, part1, 0x00);
            if (part2)
                dmaCopyVramFast(((u16)u->addr << 7), u->buffer + part1, part2, 0x00);
        }
        else
        {
            // Split column upload around VRAM ring-buffer wrap point
            u8 part1 = 128 - vram_top;
            u8 part2 = vram_top;
            dmaCopyVramFast((((u16)vram_top << 7) + u->addr), u->buffer, part1, 0x02);
            if (part2)
                dmaCopyVramFast(u->addr, u->buffer + part1, part2, 0x02);
        }
    }

    *(volatile u8*)0x2115 = 0x80;  // Restore VRAM increment mode for word writes
    update_count = 0;
}

//=========================================================
// COLLISION & DEBUG
//=========================================================

// Returns the tile ID at the given world pixel position
u8 overworldGetTileAt(s16 px, s16 py)
{
    u16 world_x = (u16)px & 4095;   // Wrap X to world bounds
    u16 world_y = (u16)py & 4095;   // Wrap Y to world bounds

    u16 tile_x  = world_x >> 3;     // Convert pixel to tile coordinate
    u16 tile_y  = world_y >> 3;

    u16 chunk_x = tile_x >> 7;      // Which 128-tile chunk column
    u16 chunk_y = tile_y >> 7;      // Which 128-tile chunk row

    u8* chunk   = chunk_table[chunk_y][chunk_x];

    u16 local_x = tile_x & 127;     // Offset within the chunk
    u16 local_y = tile_y & 127;

    return chunk[(local_y << 7) | local_x] & 0xFF;  // Return raw tile ID
}
```

---

## `airship.c`

This file contains the airship implementation and most of the advanced Mode 7 functionality. Pressing the A button activates airship mode and enables:

- world curvature effect
- distortion effects
- zoom-out behavior
- Mode 3 perspective sky

This system combines Mode 7 transformations with additional background techniques to simulate altitude and perspective.

```c
#include <snes.h>
#include "airship.h"
#include "overworld.h"

//=========================================================
// EXTERNALS
//=========================================================

extern s16 cam_x;   // Global camera X in pixels, owned by main.c
extern s16 cam_y;   // Global camera Y in pixels, owned by main.c

extern char sky_tiles, sky_tiles_end;   // Sky BG tileset binary from ROM
extern char sky_map,   sky_map_end;     // Sky BG tilemap binary from ROM
extern char sky_pal,   sky_pal_end;     // Sky BG palette binary from ROM

//=========================================================
// DEFINES
//=========================================================

#define SKYLINE_Y      31               // Scanline where the sky ends
#define GROUND_OVERLAP  3               // Lines the sky overlaps into the ground region
#define GROUND_START   (SKYLINE_Y - GROUND_OVERLAP)  // First scanline of ground/Mode7 area
#define SCREEN_H      224               // Total screen height in scanlines
#define GROUNDLINES   (SCREEN_H - SKYLINE_Y)         // Number of scanlines in the Mode7 ground area

#define SKY_TILE_ADDR 0x5000            // VRAM address for sky tileset
#define SKY_MAP_ADDR  0x6000            // VRAM address for sky tilemap

//=========================================================
// STATE
//=========================================================

AirshipState airship_state = AIRSHIP_OFF;   // Current airship mode state

static s16 sky_x = 0;   // Horizontal scroll offset for sky BG
static s16 sky_y = 0;   // Vertical scroll offset for sky BG

//=========================================================
// HDMA BUFFERS
//=========================================================

// Per-scanline Mode7 matrix values built by buildPerspective()
static u8 PerspectiveA[GROUNDLINES * 3 + 4];   // Matrix A (X scale)
static u8 PerspectiveB[GROUNDLINES * 3 + 4];   // Matrix B (X shear, kept at 0)
static u8 PerspectiveC[GROUNDLINES * 3 + 4];   // Matrix C (Y shear, kept at 0)
static u8 PerspectiveD[GROUNDLINES * 3 + 4];   // Matrix D (Y scale)

// DMA source pointers for each HDMA channel
static dmaMemory dma_mode;  // Channel 1: BG mode switch table
static dmaMemory dma_bg;    // Channel 2: TM register table
static dmaMemory dma_a;     // Channel 3: Mode7 matrix A
static dmaMemory dma_b;     // Channel 4: Mode7 matrix B
static dmaMemory dma_c;     // Channel 5: Mode7 matrix C
static dmaMemory dma_d;     // Channel 6: Mode7 matrix D

//=========================================================
// HDMA TABLES
//=========================================================

// Switches BG rendering mode at the horizon scanline
static const u8 ModeTable[5] =
{
    GROUND_START, BG_MODE3,   // Sky area: Mode3 (4bpp BG)
    1,            BG_MODE7,   // Ground area: Mode7
    0
};

// Controls which BG layers are active per region via REG_TM
const u8 BGTable[5] =
{
    GROUND_START, 0x12,   // Sky area: OBJ + BG2
    1,            0x11,   // Ground area: OBJ + BG1 (Mode7)
    0
};

//=========================================================
// FADE
//=========================================================

// Fade screen to black over 16 frames
static void fadeOut(void)
{
    u8 i;
    for (i = 0; i < 16; i++)
    {
        WaitForVBlank();
        REG_INIDISP = (15 - i) & 15;
    }
}

// Fade screen from black to full brightness over 16 frames
static void fadeIn(void)
{
    u8 i;
    for (i = 0; i < 16; i++)
    {
        WaitForVBlank();
        REG_INIDISP = i & 15;
    }
}

//=========================================================
// PERSPECTIVE BUILDER
//=========================================================

// Fills HDMA perspective buffers with per-scanline Mode7 matrix scale values
static void buildPerspective(void)
{
    u16 i;
    u16 p = 0;

    // Sky region: identity matrix (no distortion)
    PerspectiveA[p] = GROUND_START; PerspectiveA[p+1] = 0; PerspectiveA[p+2] = 1;
    PerspectiveB[p] = GROUND_START; PerspectiveB[p+1] = 0; PerspectiveB[p+2] = 0;
    PerspectiveC[p] = GROUND_START; PerspectiveC[p+1] = 0; PerspectiveC[p+2] = 0;
    PerspectiveD[p] = GROUND_START; PerspectiveD[p+1] = 0; PerspectiveD[p+2] = 1;

    p = 3;

    for (i = 0; i < GROUNDLINES; i++)
    {
        u16 z;
        s16 scale;
        s16 dist_v;
        s32 curve;

        z = (i << 8) / GROUNDLINES;  // Normalized depth 0-255 for this scanline

        // Cylindrical curve: adds convex bulge effect to the ground plane
        dist_v = (s16)z - 190;
        curve  = (s32)dist_v * dist_v;
        curve >>= 8;

        scale = 300 + (s16)curve;     // Base scale plus cylindrical correction

        scale += (256 - z) >> 3;      // Gentle horizon softening

        // Write scale to matrix A and D (uniform X/Y scale, no shear)
        PerspectiveA[p] = 1; PerspectiveA[p+1] = scale & 255; PerspectiveA[p+2] = scale >> 8;
        PerspectiveB[p] = 1; PerspectiveB[p+1] = 0;           PerspectiveB[p+2] = 0;
        PerspectiveC[p] = 1; PerspectiveC[p+1] = 0;           PerspectiveC[p+2] = 0;
        PerspectiveD[p] = 1; PerspectiveD[p+1] = scale & 255; PerspectiveD[p+2] = scale >> 8;

        p += 3;
    }

    // HDMA terminator
    PerspectiveA[p] = 0;
    PerspectiveB[p] = 0;
    PerspectiveC[p] = 0;
    PerspectiveD[p] = 0;
}

//=========================================================
// HDMA SETUP
//=========================================================

// Configures all 6 HDMA channels and enables them (channels 1-6)
static void airshipSetupHDMA(void)
{
    REG_HDMAEN = 0;  // Disable all HDMA channels before reconfiguring

    // Channel 1: write BG mode register per scanline region
    REG_DMAP1 = 0x00; REG_BBAD1 = 0x05;
    REG_A1T1LH = dma_mode.mem.c.addr; REG_A1B1 = dma_mode.mem.c.bank;

    // Channel 2: write TM (main screen designation) per scanline region
    REG_DMAP2 = 0x00; REG_BBAD2 = 0x2C;
    REG_A1T2LH = dma_bg.mem.c.addr; REG_A1B2 = dma_bg.mem.c.bank;

    // Channel 3: Mode7 matrix A (X scale), 2 bytes per scanline
    REG_DMAP3 = 0x02; REG_BBAD3 = 0x1B;
    REG_A1T3LH = dma_a.mem.c.addr; REG_A1B3 = dma_a.mem.c.bank;

    // Channel 4: Mode7 matrix B (X shear), 2 bytes per scanline
    REG_DMAP4 = 0x02; REG_BBAD4 = 0x1C;
    REG_A1T4LH = dma_b.mem.c.addr; REG_A1B4 = dma_b.mem.c.bank;

    // Channel 5: Mode7 matrix C (Y shear), 2 bytes per scanline
    REG_DMAP5 = 0x02; REG_BBAD5 = 0x1D;
    REG_A1T5LH = dma_c.mem.c.addr; REG_A1B5 = dma_c.mem.c.bank;

    // Channel 6: Mode7 matrix D (Y scale), 2 bytes per scanline
    REG_DMAP6 = 0x02; REG_BBAD6 = 0x1E;
    REG_A1T6LH = dma_d.mem.c.addr; REG_A1B6 = dma_d.mem.c.bank;

    REG_HDMAEN = 0x7E;  // Enable channels 1-6 (bits 1-6)
}

//=========================================================
// AIRSHIP ENTER
//=========================================================

// Transitions from overworld to airship/Mode7 view with fade
void airshipEnter(void)
{
    s16 sx, sy;

    fadeOut();

    buildPerspective();  // Pre-compute Mode7 scale table before uploading

    // Load sky BG2 tileset and tilemap into VRAM
    bgInitTileSet(1, &sky_tiles, &sky_pal, 0,
        (&sky_tiles_end - &sky_tiles), 16*2, BG_16COLORS, SKY_TILE_ADDR);

    bgInitMapSet(1, &sky_map, (&sky_map_end - &sky_map), SC_32x32, SKY_MAP_ADDR);

    setMode7(0);       // Switch PPU to Mode7

    REG_CGWSEL  = 0x00;  // Color math uses subscreen as second source
    REG_CGADSUB = 0x23;  // Enable color math add on BG1+BG2

    REG_M7SEL = M7_WRAP;               // Mode7 wraps at map edges
    REG_BG2SC = (SKY_MAP_ADDR >> 8);   // Point BG2 map register to sky tilemap

    // Assign source pointers for all HDMA channels
    dma_mode.mem.p = (u8*)ModeTable;
    dma_bg.mem.p   = (u8*)BGTable;
    dma_a.mem.p    = (u8*)PerspectiveA;
    dma_b.mem.p    = (u8*)PerspectiveB;
    dma_c.mem.p    = (u8*)PerspectiveC;
    dma_d.mem.p    = (u8*)PerspectiveD;

    airship_state = AIRSHIP_ON;

    overworldGetScroll(cam_x, cam_y, &sx, &sy);

    WaitForVBlank();  // Sync to VBlank before writing scroll/matrix registers

    // Apply correct camera position before screen turns on
    REG_M7HOFS = sx & 255; REG_M7HOFS = sx >> 8;
    REG_M7VOFS = sy & 255; REG_M7VOFS = sy >> 8;
    REG_M7X = (sx + 128) & 255; REG_M7X = (sx + 128) >> 8;
    REG_M7Y = (sy + 112) & 255; REG_M7Y = (sy + 112) >> 8;

    airshipSetupHDMA();  // Configure and enable all HDMA channels

    fadeIn();
}

//=========================================================
// AIRSHIP EXIT
//=========================================================

// Returns from airship view to overworld with fade
void airshipExit(void)
{
    fadeOut();

    REG_HDMAEN = 0;         // Disable all HDMA channels

    airship_state = AIRSHIP_OFF;

    overworldRestoreZoom(); // Restore Mode7 scale to overworld zoom factor

    fadeIn();
}

//=========================================================
// AIRSHIP FLUSH (called every frame inside VBlank)
//=========================================================

// Updates Mode7 scroll registers and sky BG scroll each frame
void airshipFlush(s16 cam_x, s16 cam_y)
{
    s16 sx, sy;

    if (airship_state == AIRSHIP_OFF)
        return;

    // Slow parallax scroll for sky BG
    sky_x = cam_x >> 2;
    sky_y = (cam_y >> 4) + 20;

    overworldGetScroll(cam_x, cam_y, &sx, &sy);

    // Update Mode7 scroll and rotation center registers
    REG_M7HOFS = sx & 255; REG_M7HOFS = sx >> 8;
    REG_M7VOFS = sy & 255; REG_M7VOFS = sy >> 8;
    REG_M7X = (sx + 128) & 255; REG_M7X = (sx + 128) >> 8;
    REG_M7Y = (sy + 112) & 255; REG_M7Y = (sy + 112) >> 8;

    // Apply sky BG horizontal scroll
    REG_BG2HOFS = (sky_x & 255);
    REG_BG2HOFS = (sky_x >> 8) & 255;
}
```

---

## `player.c`

This file contains the collision system. The `BLOCKED_TILES` array is a hardcoded list of tile IDs considered solid. Any tile ID present in this list blocks movement.

`overworldGetTileAt(px, py)` receives world coordinates in pixels and returns the tile ID at that location. Movement occurs in discrete 8-pixel steps. Collision is checked only once at the beginning of movement.

- If the destination is valid: `move_rem` is set to `7` and the player continues moving during subsequent frames.
- If blocked: movement does not occur.

**Logic flow:**

```
playerUpdate()
 └─ directional key pressed, move_rem == 0
      └─ calculate nx, ny
           └─ isSolid(corner1)||isSolid(corner2)
                └─ overworldGetTileAt(px,py)
                     └─ chunk_table[chunk_y][chunk_x][local_y*128+local_x]
                          └─ returns tile ID
                └─ scan BLOCKED_TILES[]
                     └─ return 1 or 0
      └─ if free:
            move_rem=7
      └─ if blocked:
            do nothing
```

Although the structure appears elaborate, the system is fundamentally based on a simple flag-driven movement state.

```c
#include <snes.h>
#include "player.h"
#include "res/gfx/Ceodore_meta.inc"
#include "res/gfx/Ceodore.inc"

extern s16 cam_x;   // Global camera X in pixels, owned by main.c
extern s16 cam_y;   // Global camera Y in pixels, owned by main.c

//=========================================================
// DEFINES
//=========================================================

#define STEP_SIZE    8                          // Movement distance per step in pixels
#define STEP_SPEED   1                          // Pixels moved per frame during a step
#define BLOCKED_COUNT (sizeof(BLOCKED_TILES))   // Number of solid tile IDs in the collision table

//=========================================================
// STATE
//=========================================================

static u8  playerDir;     // Visual/input direction (last pressed direction key)
static u8  moveDir;       // Locked movement direction for the current step
static u8  animFrame;     // Current animation frame index (0 or 1)
static u8  animTimer;     // Counts frames to control animation speed
static u8  isMoving;      // 1 if the player is currently moving, 0 if idle
static u16 currentFrame;  // Index into the metasprite table for the current pose
static s16 move_rem = 0;  // Remaining pixels left in the current 8-pixel step

//=========================================================
// COLLISION TABLE
//=========================================================

extern u8 overworldGetTileAt(s16 px, s16 py);  // Returns tile ID at world pixel position

// Tile IDs that block player movement
const u8 BLOCKED_TILES[] =
{
    0x04,0x05,0x06,0x08,0x09,0x0C,0x0D,

    0x10,0x11,0x13,0x14,0x15,0x16,0x17,
    0x18,0x19,0x1A,0x1C,0x1D,0x1E,0x1F,

    0x20,0x22,0x23,0x26,0x27,0x28,0x29,
    0x2A,0x2B,0x2C,

    0x33,0x35,0x36,0x39,0x3A,0x3B,
    0x3C,0x3E,0x3F,

    0x42,

    0x67,0x6D,

    0x90,0x91,0x95,0x96,0x97,
    0x98,0x99,0x9A,

    0xA1,0xA2,0xA3,

    0xBF,

    0xE7,0xE8
};

// Returns 1 if the tile at pixel position (px, py) is in the blocked list
static u8 isSolid(s16 px, s16 py)
{
    u8 tile = overworldGetTileAt(px, py);
    u8 i;
    for (i = 0; i < BLOCKED_COUNT; i++)
        if (tile == BLOCKED_TILES[i])
            return 1;
    return 0;
}

//=========================================================
// DEBUG
//=========================================================

// Writes surrounding tile IDs to WRAM for external debugging tools
static void debugUpdateWramTiles(s16 player_x, s16 player_y)
{
    volatile u8* wram_atual    = (volatile u8*)0x03AD;
    volatile u8* wram_direita  = (volatile u8*)0x03AE;
    volatile u8* wram_esquerda = (volatile u8*)0x03AF;
    volatile u8* wram_cima     = (volatile u8*)0x03B0;
    volatile u8* wram_baixo    = (volatile u8*)0x03B1;

    *wram_atual    = overworldGetTileAt(player_x,     player_y);
    *wram_direita  = overworldGetTileAt(player_x + 8, player_y);
    *wram_esquerda = overworldGetTileAt(player_x - 8, player_y);
    *wram_cima     = overworldGetTileAt(player_x,     player_y - 8);
    *wram_baixo    = overworldGetTileAt(player_x,     player_y + 8);
}

//=========================================================
// ANIMATION
//=========================================================

// Selects the correct metasprite frame based on movement state and direction
static void playerUpdateAnim(void)
{
    u8 visualDir = isMoving ? moveDir : playerDir;

    if (!isMoving)
    {
        // Pick idle frame for current facing direction
        switch (visualDir)
        {
            case DIR_DOWN:  currentFrame = FRAME_FRONT_IDLE;      break;
            case DIR_UP:    currentFrame = FRAME_BACK_IDLE;       break;
            case DIR_LEFT:  currentFrame = FRAME_SIDE_IDLE_LEFT;  break;
            case DIR_RIGHT: currentFrame = FRAME_SIDE_IDLE_RIGHT; break;
        }
        return;
    }

    // Advance walk cycle every 10 frames
    animTimer++;
    if (animTimer > 10)
    {
        animTimer = 0;
        animFrame ^= 1;  // Toggle between frame 0 and 1
    }

    // Pick walk or idle frame alternating for walk cycle
    switch (visualDir)
    {
        case DIR_DOWN:
            currentFrame = animFrame ? FRAME_FRONT_WALK      : FRAME_FRONT_IDLE;      break;
        case DIR_UP:
            currentFrame = animFrame ? FRAME_BACK_WALK       : FRAME_BACK_IDLE;       break;
        case DIR_LEFT:
            currentFrame = animFrame ? FRAME_SIDE_WALK1_LEFT : FRAME_SIDE_WALK2_LEFT; break;
        case DIR_RIGHT:
            currentFrame = animFrame ? FRAME_SIDE_WALK1_RIGHT: FRAME_SIDE_WALK2_RIGHT;break;
    }
}

//=========================================================
// INIT
//=========================================================

// Sets initial player state and uploads sprite tiles and palette to VRAM
void playerInit(s16 startX, s16 startY)
{
    cam_x = startX;
    cam_y = startY;

    playerDir    = DIR_DOWN;
    moveDir      = DIR_DOWN;
    animFrame    = 0;
    animTimer    = 0;
    isMoving     = 0;
    currentFrame = FRAME_FRONT_IDLE;

    dmaCopyVram((u8*)&ceodore_tiles, 0x4000,
        (u16)(&ceodore_tiles_end - &ceodore_tiles));  // Upload sprite sheet to VRAM

    setPalette((u8*)&ceodore_pal, 128, 32);           // Load sprite palette into CGRAM slot 8
}

//=========================================================
// INPUT DIRECTION RESOLVER
//=========================================================

// Resolves the active direction from newly pressed and currently held buttons
static void playerUpdateDirection(u16 held, u16 press)
{
    // Newly pressed button takes priority over held buttons
    if      (press & KEY_UP)    playerDir = DIR_UP;
    else if (press & KEY_DOWN)  playerDir = DIR_DOWN;
    else if (press & KEY_LEFT)  playerDir = DIR_LEFT;
    else if (press & KEY_RIGHT) playerDir = DIR_RIGHT;

    // If current direction key is released, fall back to another held key
    switch (playerDir)
    {
        case DIR_UP:
            if      (!(held & KEY_UP)    && (held & KEY_DOWN))  playerDir = DIR_DOWN;
            else if (!(held & KEY_UP)    && (held & KEY_LEFT))  playerDir = DIR_LEFT;
            else if (!(held & KEY_UP)    && (held & KEY_RIGHT)) playerDir = DIR_RIGHT;
            break;
        case DIR_DOWN:
            if      (!(held & KEY_DOWN)  && (held & KEY_UP))    playerDir = DIR_UP;
            else if (!(held & KEY_DOWN)  && (held & KEY_LEFT))  playerDir = DIR_LEFT;
            else if (!(held & KEY_DOWN)  && (held & KEY_RIGHT)) playerDir = DIR_RIGHT;
            break;
        case DIR_LEFT:
            if      (!(held & KEY_LEFT)  && (held & KEY_UP))    playerDir = DIR_UP;
            else if (!(held & KEY_LEFT)  && (held & KEY_DOWN))  playerDir = DIR_DOWN;
            else if (!(held & KEY_LEFT)  && (held & KEY_RIGHT)) playerDir = DIR_RIGHT;
            break;
        case DIR_RIGHT:
            if      (!(held & KEY_RIGHT) && (held & KEY_UP))    playerDir = DIR_UP;
            else if (!(held & KEY_RIGHT) && (held & KEY_DOWN))  playerDir = DIR_DOWN;
            else if (!(held & KEY_RIGHT) && (held & KEY_LEFT))  playerDir = DIR_LEFT;
            break;
    }
}

//=========================================================
// UPDATE (called once per frame before VBlank)
//=========================================================

void playerUpdate(void)
{
    u16 held  = padsCurrent(0);  // All currently held buttons
    u16 press = padsDown(0);     // Buttons pressed this frame only

    isMoving = 0;

    playerUpdateDirection(held, press);

    //-----------------------------------------------------
    // Continue current in-progress step
    //-----------------------------------------------------

    if (move_rem > 0)
    {
        isMoving = 1;
        switch (moveDir)
        {
            case DIR_UP:    cam_y -= STEP_SPEED; break;
            case DIR_DOWN:  cam_y += STEP_SPEED; break;
            case DIR_LEFT:  cam_x -= STEP_SPEED; break;
            case DIR_RIGHT: cam_x += STEP_SPEED; break;
        }
        move_rem -= STEP_SPEED;
    }

    //-----------------------------------------------------
    // Start a new step if a direction is held and no step is active
    //-----------------------------------------------------

    else if (held & (KEY_UP | KEY_DOWN | KEY_LEFT | KEY_RIGHT))
    {
        moveDir = playerDir;  // Lock direction for the full 8-pixel step

        s16 px = (cam_x + 128) & 4095;  // Player center X in world space
        s16 py = (cam_y + 128) & 4095;  // Player center Y in world space

        s16 nx = px;
        s16 ny = py;

        // Compute target position after one step
        switch (moveDir)
        {
            case DIR_UP:    ny = (py - STEP_SIZE) & 4095; break;
            case DIR_DOWN:  ny = (py + STEP_SIZE) & 4095; break;
            case DIR_LEFT:  nx = (px - STEP_SIZE) & 4095; break;
            case DIR_RIGHT: nx = (px + STEP_SIZE) & 4095; break;
        }

        // Check two corner points ahead in the movement direction
        u8 bloqueado = 0;
        switch (moveDir)
        {
            case DIR_UP:
                bloqueado = isSolid((nx-4)&4095,(ny-4)&4095) || isSolid((nx+3)&4095,(ny-4)&4095);
                break;
            case DIR_DOWN:
                bloqueado = isSolid((nx-4)&4095,(ny+3)&4095) || isSolid((nx+3)&4095,(ny+3)&4095);
                break;
            case DIR_LEFT:
                bloqueado = isSolid((nx-4)&4095,(ny-4)&4095) || isSolid((nx-4)&4095,(ny+3)&4095);
                break;
            case DIR_RIGHT:
                bloqueado = isSolid((nx+3)&4095,(ny-4)&4095) || isSolid((nx+3)&4095,(ny+3)&4095);
                break;
        }

        if (!bloqueado)
        {
            isMoving = 1;
            move_rem = STEP_SIZE - STEP_SPEED;  // Remaining pixels after first frame of step
            switch (moveDir)
            {
                case DIR_UP:    cam_y -= STEP_SPEED; break;
                case DIR_DOWN:  cam_y += STEP_SPEED; break;
                case DIR_LEFT:  cam_x -= STEP_SPEED; break;
                case DIR_RIGHT: cam_x += STEP_SPEED; break;
            }
        }
    }

    // Wrap camera position within 4096x4096 world bounds
    while (cam_x < 0)     cam_x += 4096;
    while (cam_x >= 4096) cam_x -= 4096;
    while (cam_y < 0)     cam_y += 4096;
    while (cam_y >= 4096) cam_y -= 4096;

    playerUpdateAnim();  // Select animation frame based on current movement state

    // Debug: write adjacent tile IDs to WRAM for inspection
    s16 player_x = (cam_x + 128) & 4095;
    s16 player_y = (cam_y + 128) & 4095;
    debugUpdateWramTiles(player_x, player_y);
}

//=========================================================
// DRAW (called once per frame before VBlank)
//=========================================================

// Writes the two OAM entries that form the 16x32 metasprite
void playerDraw(void)
{
    u8 screenX = 128 - 8;   // Center sprite horizontally on screen midpoint
    u8 screenY = 112 - 16;  // Center sprite vertically (16px offset for 32px tall sprite)

    const t_metasprite *meta = Ceodore_metasprites[currentFrame];

    u8 visualDir = isMoving ? moveDir : playerDir;
    u8 flipH     = (visualDir == DIR_RIGHT) ? 1 : 0;  // Mirror sprite for rightward movement

    // Top tile of the metasprite (upper 16x16)
    oamSet(0, screenX, screenY, 2, flipH, 0, meta[0].dtile, 8);
    oamSetEx(0, OBJ_SMALL, OBJ_SHOW);

    // Bottom tile of the metasprite (lower 16x16)
    oamSet(4, screenX, screenY + 16, 2, flipH, 0, meta[1].dtile, 8);
    oamSetEx(4, OBJ_SMALL, OBJ_SHOW);
}
```

---

## `main.c`

This file handles the high-level game flow and state transitions. Its primary responsibility is determining whether the game is currently operating in airship mode and switching behavior accordingly.

```c
#include <snes.h>
#include "src/player.h"
#include "src/overworld.h"
#include "src/airship.h"
#include "src/airship_subscreen.h"

//=========================================================
// EXTERNALS
//=========================================================

extern u16 m7sx, m7sy;  // Mode7 scale registers exposed by PVSNESLib

//=========================================================
// GLOBALS
//=========================================================

s16 cam_x = 1576;       // Initial camera X position in pixels
s16 cam_y = 2176;       // Initial camera Y position in pixels
u16 zoom  = 0x0100;     // Mode7 zoom factor (0x0100 = 1:1 scale)

//=========================================================
// ENTRY POINT
//=========================================================

int main(void)
{
    // Initialize OAM with small=16x16 large=32x32, tiles at 0x4000, large at 0x6000
    oamInitDynamicSprite(0x4000, 0x6000, 0, 0, OBJ_SIZE16_L32);

    // Set Mode7 scale to default zoom
    m7sx = zoom;
    m7sy = zoom;

    // Load tileset, map chunks, palette and do initial full VRAM upload
    overworldInit(cam_x, cam_y, zoom);

    setScreenOn();
    REG_INIDISP = 0x80;  // Screen off during player asset loading

    // Upload player tiles and palette to VRAM
    playerInit(cam_x, cam_y);

    REG_INIDISP = 0x0F;  // Full brightness, screen on

    //=========================================================
    // MAIN LOOP
    //=========================================================

    while (1)
    {
        //-----------------------------------------------------
        // CPU STAGE — logic before VBlank
        //-----------------------------------------------------

        playerUpdate();                   // Read input and update player position/animation

        u16 pad = padsCurrent(0);

        if ((pad & KEY_A) && airship_state == AIRSHIP_OFF)
            airshipEnter();               // Transition into airship/Mode7 view

        if ((pad & KEY_B) && airship_state == AIRSHIP_ON)
            airshipExit();                // Return to overworld view

        playerDraw();                     // Write sprite entries into OAM shadow buffer

        overworldPrepare(cam_x, cam_y);   // Build pending row/column updates for scroll
        overworldAnimateWaterTick();      // Advance water animation frame in RAM

        //-----------------------------------------------------
        // VBLANK STAGE — hardware writes inside VBlank window
        //-----------------------------------------------------

        WaitForVBlank();

        oamUpdate();                      // DMA OAM shadow buffer to hardware OAM
        overworldFlush();                 // DMA pending tile row/column updates to VRAM
        overworldScroll(cam_x, cam_y);    // Update BG scroll registers for current camera

        if (water_timer == 0)
            overworldAnimateWaterFlush(); // DMA animated water tiles to VRAM every 8 frames

        airshipFlush(cam_x, cam_y);       // Update Mode7 matrix and sky scroll registers
        airshipSubFlush();                // Repoint HDMA channel 7 gradient table each frame
    }

    return 0;
}
```

---


