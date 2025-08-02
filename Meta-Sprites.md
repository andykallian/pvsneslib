A metasprite is a larger sprite made up from a collection of smaller individual hardware sprites.  

Different frames of the same metasprites can share tile data. All together represent one single object.  

## Metasprite support  

The api supports metasprites with 32x32, 16x16 and 8x8 sprites width and height.  you **can't** mixed different sizes for one metasprite.

It works with sprites mode compatible with them (not the 64x64 modes, check [this page](https://github.com/alekmaul/pvsneslib/wiki/Sprites#sprite-sizes)).  

Use gfx4snes tool to convert single or multiple frames of graphics into metasprite structured data for use with the **oamMetaDrawD...()** functions.

Metasprites can be used with dynamisc sprite engine or not. If you do not want to use dynamic sprite engine, you will have to load in Vram the tiles of your sprites before.  

## Metasprite definitions

gfx4snes will generate a file which contains the metasprite definition. It is a constant array of **t_metasprite** structs. 

```
const metasprite_t spritehero_metasprite0[] = {
	METASPR_ITEM(0, 0, 0, OBJ_PAL(0) | OBJ_PRIO(2)),
	METASPR_ITEM(16, 0, 1, OBJ_PAL(0) | OBJ_PRIO(2)),
	METASPR_ITEM(0, 16, 2, OBJ_PAL(0) | OBJ_PRIO(2)),
	METASPR_ITEM(16, 16, 3, OBJ_PAL(0) | OBJ_PRIO(2)),
	METASPR_ITEM(0, 32, 4, OBJ_PAL(0) | OBJ_PRIO(2)),
	METASPR_ITEM(16, 32, 5, OBJ_PAL(0) | OBJ_PRIO(2)),
	METASPR_TERM
};
```

Each **t_metasprite** item has 4 fields, it will end with a word value of **-128** in the dx field (named **METASPR_TERM**).  

```
u16 dx, dy       X & Y coordinates of the sprite relative to the metasprite origin (0,0)
u16 dtile        Start tile relative to the metasprites own set of tiles
u8 props         Property Flags (palette and priority, flip x/y will be added later)
```

## Converting with gfx4snes

You can use **gfx4snes**, shipped with **devkitsnes** to convert your bitmap files into a correct format for PVSnesLib. Remember that the size must be a multiple of 8 pixels 8x8, 16x16 or 32x32.  

You can of course put more than one metasprite in the same graphic file.  

Here is an example of a makefile instruction to convert a metasprite of 16 pix width / height with **gfx4snes**.  

```
#---------------------------------------------------------------------------------
spritehero.pic: spritehero.png
	@echo convert meta sprites as 16px  ... $(notdir $@)
	$(GFXCONV) -s 16 -o 16 -u 16 -T -X 32 -Y 48 -P 2 -i $<
```
 
  **s 16** because we have 16 pix width sprite mode
  **o 16** because we are going to use only one palette of 16 colors  
  **u 16** because we are going to use the 16 colors mode  
  **-T** to create the metasprite structure
  **-X 32** because our metasprite is 32 pixels width
  **-Y 48** because our metasprite is 48 pixels height
  
Then, create a **data.asm** file with the converted file include in it, like you can see in **PVSnesLib** examples. This file will be included with your project and linked with the graphics.  

```
.include "hdr.asm"

.section ".rodata3" superfree

.include "spritehero_data.as"

.ends
```

Also, add the metasprite definitions inside your C source file, same as the external references for the graphics you added to **data.asm** file.  

During initialization process in your C files, you will have to declare some external variables to allow functions to know which graphic sprites you are going to use, regarding the name you entered in your **data.asm** file.  

```
#include "spritehero_meta.inc"
#include "spritehero.inc"
```

## Playing with dynamic engine and metasprites  

In your main C file, initialize the dynamic sprite engine with the correct avlue for large and small sprite mode.  

In our example, we use 0x0000 for large sprites and 0x1000 for small ones, with the sprites 32x32 and 16x16 for large and small sprites.  
We also need to add the refresh flag to the 1st oambuffer entry to allow the dynamic engine to store the first graphics in vram.  

```
    // Init sprite engine (0x0000 for large, 0x1000 for small)
    oamInitDynamicSprite(0x0000, 0x1000, 0, 0, OBJ_SIZE16_L32);

	// Put default refresh flag for 1st sprite (will refresh all metasprites)
	//  other flags are set by metasprite function
	oambuffer[1].oamrefresh=1;
```

We also need to load the palette in Vram for the complete metasprite structure.  

```
    // Init Sprites pal
    setPalette(&spritehero_pal, 128 + 0 * 16, 16 * 2);
```

When you want to display your meta sprite, you will just need to refer to the correct index of sprite (**1** in our example, the coordinates to the metapsrite, the aaddress of the graphics and the fact we need to use small or large sprites.  

Regarding our example, as the metasprite is made of **16x16** sprites and the pritemode is **OBJSIZE16_L32, we will have to specify PVSlesLib to use the small table entry for this one.  

```
    // draw the sprite
    oamMetaDrawDyn16(1, 64,160, (u8 *) spritehero_metasprites[0],(u8 *) &spritehero_til, OBJ_SMALL);
```

## Playing with static graphics for metasprites  

The only differences with Dynamic engine is that you do not need to initialize the engine but copy the graphics to Vram at the address you want.  


```
    // Init Sprites pal & graphics
    setPalette(&spritehero_pal, 128 + 0 * 16, 16 * 2);

    dmaCopyVram((u8 *) &spritehero_til,0x1000, (&spritehero_tilend - &spritehero_til));
```

Then when you want to display a metasprite, you have to use the function **oamMetaDraw16**.

```
    // draw the sprite
    oamMetaDraw16(1, 64,160, (u8 *) spritehero_metasprites[0], OBJ_SMALL);
```

See the Dynamic Engine meta sprite and Metasprite examples shipped with PVSnesLib for the complete source code.
