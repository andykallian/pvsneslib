Background is a term in 2D graphics programming which refers to the image displayed on background of screen, in opposite of sprites. The Snes has dedicated hardware for dealing with multiple types of backgrounds.

## SNES Background Specs

SNES backgrounds are composed of tiled background. A tiled background consists of a collection of tiles and a map. Each tile has a number , so the map is the collection of numbers that are represented with the tiles.

A tile is a collection of 64 pixels forming a rectangle 8 pixels wide and 8 pixels tall. It is also possible to have 16x16 tiles but it is not implemented in PVSneslib for now.
A tile’s colors are defined by the tile’s color palette.

At least, a palette is a collection of either 4, 16 or 256 colors.

In order to render background, you has to set few things for each background, remember to set this for every background you want to render.

## Background Mode (affects number of planes, bitdepth)

The SNES has 8 background modes.
- Named Mode 0-7
- Mode 1 has a variation mode
- Mode 7 has a submode called Mode 7 Ext. BG
- Each mode has a different amount of background planes available
- Each mode has a different amount of colors for the available background planes for that mode
- Each mode has slightly different rendering properties
- For all modes and for all graphics planes, color index 0 for any subpalette is the transparency color.

```
Mode    # Colors for BG
         1   2   3   4
======---=---=---=---=
0        4   4   4   4
1       16  16   4   -
2       16  16   -   -
3      256  16   -   -
4      256   4   -   -
5       16   4   -   -
6       16   -   -   -
7      256   -   -   -
7EXTBG 256 128   -   -
```
In all modes and for all BGs, **color 0 in any palette is considered transparent**.

## Background sizes

In VRAM there is a two-dimensional array that is a map of the tiles on the screen. Depending on the setting of registers $2107 to $210A, this map may be 32x32, 32x64, 64x32 or 64x64 tiles in size.

All tilemaps are 32x32, bits 0-1 simply select the number of 32x32 tilemaps and how they're laid out in memory.

```
00  32x32   AA
            AA
01  64x32   AB
            AB
10  32x64   AA
            BB
11  64x64   AB
            CD
```

## Background in VRAM

Each entry of the background map is a 16 bits value.  
```
High     Low          Legend->  c: Starting character (tile) number
vhopppcc cccccccc               h: horizontal flip  v: vertical flip
                                p: palette number   o: priority bit
```
The character number indexes into an "array of tiles" starting at a base VRAM location selected by registers $210B or $210C.  
The byte address in VRAM where the character data starts can be found using the following calculation:  
```
    address_of_character = (base_location_bits << 13) + (8 * color_depth * character_number);
```
The horizontal and vertical flip bits, if set to 1, will cause the characters to be mirrored when shown on the screen, so that they are facing the opposite direction. When in 16x16 tile mode, the entire tile is flipped (pixel 0 is swapped with pixel 15) rather than the individual 8x8 sub tiles being flipped.  
  
Addresses of brakcgrounds are in units of 4 KB **(value × $1000)**  

The priority bit has the effect of deciding whether a given tile is 'on top of' or behind other BGs and sprites.  For more information on the drawing order, see bit 3 of register $2105.  

## Backgrounds with PVSnesLib
The most commonly used mode is Mode 1, which has two main 4bpp (16 color) layers BG1 and BG2, and one auxiliar 2bpp (4 color) layer BG3.  
BG3 has an additional priority control in mode 1. Its priority bit in BGMODE allows it to be rendered either above or below BG1 and BG2.  
BG3 selects a palette from the first 32 entries of CGRAM.  
In many games, BG1 and BG2 are used for a colourful main background with parallax, and BG3 to overlay a HUD or text box.  
BG3 can also be useful for things like a blended cloud or fog in the foreground, or a third parallax layer in the deep background (Super Metroid has many good examples).

To display a background with PVSnesLib, you must create a bitmap with a maximum of 16 color palettes per tile of 8x8 pixels and also no more than 8 palettes.  

Also, you need to understand priorities in mode 1 regarding backgrounds and sprites.  

```
BG3 tiles with priority 1 if bit 3 of $2105 is set
Sprites with priority 3
BG1 tiles with priority 1
BG2 tiles with priority 1
Sprites with priority 2
BG1 tiles with priority 0
BG2 tiles with priority 0
Sprites with priority 1
BG3 tiles with priority 1 if bit 3 of $2105 is clear
Sprites with priority 0
BG3 tiles with priority 0
```


rilden did a tool that can easily convert bitmap with the correct number of colors and palette. You can check it [here](https://rilden.github.io/tiledpalettequant/) and the source code is available [here](https://github.com/rilden/tiledpalettequant).  

## Converting with gfx4snes

You can use **gfx4snes**, shipped with **devkitsnes** to convert your bitmap files into a correct format for PVSnesLib. 

Here is an example of a makefile instruction to convert a background with only 1 palette of 16 colors with **gfx4snes**.  

```
pvsneslib.pic: pvsneslib.png
	@echo convert bmp ... $(notdir $@)
	$(GFXCONV) -s 8 -o 16 -u 16 -e 0 -p -m -i $<
```

  **s 8** because we have 8 pix width  & height for the tiles   
  **o 16** because we are going to use only one palette of 16 colors  
  **u 16** because we are going to use the 16 colors mode  
  **e 0** because we are using the first palette entry  
  **p** to export the palette and **m** to export the map attributes  

Then, create a **data.asm** file with the converted file include in it, like you can see in **PVSnesLib** examples. This file will be included with your project and linked with the graphics. We are using 2 banks because it will not fit only in one bank of 32KB.  

```
.section ".rodata1" superfree

patterns:
.incbin "pvsneslib.pic"
patterns_end:

.ends

.section ".rodata2" superfree
map:
.incbin "pvsneslib.map"
map_end:

palette:
.incbin "pvsneslib.pal"
palette_end:
```

During initialization process in your C files, you will have to declare some external variables to allow functions to know which graphic backgrounds you are going to use, regarding the name you entered in your data.asm file.  

```
extern char patterns, patterns_end;
extern char palette, palette_end;
extern char map, map_end;
```

## Init the background

In order to have the correct tiles for the background, your need to init it with the **bgInitTileSet** function. It will initialize the patterns of the tiles and the palette.  
As you can see, we also specify which VRAM address we want to use for this background, **0x4000** in our example.

The trick **(&patterns_end - &patterns)** is used to calculate the size of the patterns, same thing for the palette with ** (&palette_end - &palette)**.  

```
    // Copy tiles to VRAM
    bgInitTileSet(0, &patterns, &palette, 0, (&patterns_end - &patterns), (&palette_end - &palette), BG_16COLORS, 0x4000);
```

The last parameter to explain is the number of colors used by the background. **BG_16COLORS** in our example.

```
Color Mode   Description
============ ======================
BG_4COLORS0  For mode 0 with 4 colors per background
BG_16COLORS  For mode 1,2,5,6 with 16 colors per background
BG_256COLORS For mode 3,4,7 with 256 colors per background
```

## Displaying the background

Backgrounds begin at x = 0 and y = 1. It is not a bug and it is linked to a technical constraint :

> Note that many games will set their vertical scroll values to -1 rather than 0.
> This is because the SNES loads OBJ data for each scanline during the previous scanline. The very first line, though, wouldn't have any OBJ data loaded! So the SNES doesn't actually output scanline 0, although it does everything to render it.

If you want more information on it, you can consult [this page](https://wiki.superfamicom.org/backgrounds#toc-3)

After sending pattern and palette to VRAM, we need to send the map attribute to VRAM.  

**SC_32x32** if the size of the map, it can be SC_32x32,SC_64x32, SC_32x64 or SC_64x64 regarding the size of the background you need.  

**0x0000** is the VRAM address where you want to put the map attribute.  

```
    // Copy Map to VRAM
    bgInitMapSet(0, &map, (&map_end - &map), SC_32x32, 0x0000);
```
Then, we need to activate the display with the correct backgrounds enabled or not.  

```
    // Now Put in 16 color mode and disable other BGs except 1st one
    setMode(BG_MODE1, 0);
    bgSetDisable(1);
    bgSetDisable(2);
    setScreenOn();
```

It is not mandatory to use **setMode** and **bgSetEnable/bgSetDisable** each time if you're going to display more than one background with the same configuration. Just put the screen off, change the patterns / map with **dmaCopyVram**, for example, and then put again the screen on.  

Check [PVSnesLib Background examples](https://github.com/alekmaul/pvsneslib/tree/master/snes-examples/graphics/Backgrounds) to understand how to manage backgrounds with all the SNES modes.  

**That's all for background explanation, you are now able to display some screens on SNES !**  

