A dynamic sprite is a sprite where graphics are dynamically uploads to video ram (VRAM). So only one frame of the sprite is in VRAM at a given moment.  
More specifically, PVSneslib allows dynamic uploads to VRAM with sprite functions, allowing you basically as many frames in a sprite as you like.  

![dynamicsprite_tut00](https://github.com/user-attachments/assets/d37edb98-c81e-4956-8075-1177eb5bba5e)

## Dynamic sprites support  

Please refere to Sprite page before, to understand how the SNES and PVSNesLib support sprites, check [this page](https://github.com/alekmaul/pvsneslib/wiki/Sprites).  

PVSnesLib supports all sprite sizes for dynamic sprites, as they are 'normal' sprites, the only thing is to declare the correct.  

## Dynamic sprite structure  

The structure uses for dynamic sprite is named **oambuffer**.  

It contains all the sprite definitions for a specific sprite. It has 128 entries, as the number of sprites for the SNES.  

```
    s16 oamx;        x position on the screen  
    s16 oamy;        y position on the screen  
    u16 oamframeid;  frame index in graphic file of the sprite  
    u8 oamattribute; sprite attribute value (vhoopppc vh : vertical/horizontal flip o: priority bits p: palette num c : last byte of tile num)  
    u8 oamrefresh;   =1 if we need to load graphics from graphic file  
    u8 *oamgraphics; pointer to graphic file   
    u16 dummy1;      to be 16 aligned  
    u16 dummy2;
```

Here is an example of declaration for the first entry of a dynamic sprite.  

```
t_sprites *psprmen;                                                 // ptr to sprite 

...

    psprmen=&oambuffer[0];
    psprmen->oamx = 128-8;
    psprmen->oamy = 112-8;
    psprmen->oamattribute = OBJ_PAL(0); // 	16x16 main player and palette0 for new graphics  byte OBJ*4+3: vhoopppN
    psprmen->oamgraphics=&sprites_til;
```

## Converting with gfx4snes

You can use **gfx4snes**, shipped with **devkitsnes** to convert your bitmap files into a correct format for PVSnesLib. Remember that the size must be a multiple of 8 pixels 8x8, 16x16, 32x32 or 64x64.  

<img width="64" height="16" alt="dynamicsprite_tut01" src="https://github.com/user-attachments/assets/b1e71ec9-8d49-4837-9059-651d069fd5b8" />

You can of course put more than one sprite in the same graphic file, but as dynamic sprite also has the pointer to graphic files, you can delcar more animation in multiple files.  

Here is an example of a makefile instruction to convert a sprite of 16 pix width / height with **gfx4snes**.  

```
#---------------------------------------------------------------------------------
sprites.pic: sprites.bmp
	@echo convert bitmap ... $(notdir $@)
	$(GFXCONV) -s 16 -o 16 -u 16 -t bmp -i $<
```

  **s 16** because we have 16 pix width   
  **o 16** because we are going to use only one palette of 16 colors  
  **u 16** because we are going to use the 16 colors mode  
  
Then, create a **data.asm** file with the converted file include in it, like you can see in **PVSnesLib** examples. This file will be included with your project and linked with the graphics. **gfx4snes** helps you and create the definitions in a **xxx_data.as** file.    

```
.include "hdr.asm"

.section ".rodata1" superfree
.include "sprites_data.as"
.ends
```

During initialization process in your C files, you will have to declare some external variables to allow functions to know which graphic sprites you are going to use, regarding the name used in your **data.asm** file. Again, **gfx4snes** helps you and create the external definitions in a **xxx.inc** file.    

```
#include "sprites.inc"
```

## Playing with dynamic engine and metasprites  

In your main C file, initialize the dynamic sprite engine with the correct value for large and small sprite mode.  

In our example, we use 0x0000 for large sprites and 0x1000 for small ones, with the sprites 16x16 and 8x8 for large and small sprites.  
We also need to add the refresh flag to the 1st oambuffer entry to allow the dynamic engine to store the first graphics in vram.  

```
    // Init sprite engine (0x0000 for large, 0x1000 for small)
    oamInitDynamicSprite(0x0000, 0x1000, 0, 0, OBJ_SIZE8_L16);
```

We also need to load the palette in Vram for the dynamic sprite.  

```
    // Init Sprites pal
    setPalette(&sprites_pal, 128 + 0 * 16, 16 * 2);
```

When you want to display your dynamic sprite, you will just need to refer to the correct index of sprite (**0** in our example, the coordinates to the metdynamic sprite, the address of the graphics and the fact we need to use small or large sprites.  

Regarding our example, as the sprite is made of **16x16** sprites and the spritemode is **OBJSIZE8_L16**, we will have to specify PVSnesLib to use the small table entry for this one. We also use a pointer to address entre **#0** but it is not mandatory, it's just a question of clarity for our code.    

```
t_sprites *psprmen;                             // ptr to sprite 

...

    // prepare sprite
    psprmen=&oambuffer[0];
    psprmen->oamx = 128-8;
    psprmen->oamy = 112-8;
    psprmen->oamframeid = 0;
    psprmen->oamattribute = OBJ_PAL(0); // 	16x16 main player and palette0 for new graphics  byte OBJ*4+3: vhoopppN
    psprmen->oamgraphics=&sprites_til;
```
## Changing frame of the sprite  

The frame is stored in the **oamframeid** attribute. regarding our example, it can be a value from 0 to 3. When you update it, you must put the flag **oamrefresh** to 1 to allow the Dynamic engine to change the graphics at the end of the frame (see below).  

```
    psprmen->oamframeid++;
    psprmen->oamframeid &=3;
    psprmen->oamrefresh = 1;
```

## Refresh the sprites  

The sprite must be refresh at each frame, with the call of **oamInitDynamicSpriteEndFrame** function at the end of your current frame process.  

```
    // prepare next frame and wait vblank
    oamInitDynamicSpriteEndFrame();
    WaitForVBlank();
    oamVramQueueUpdate();

```

As you see, you also have to update VRAM with a call of **oamVramQueueUpdate** function.  

See the Dynamic Engine sprite examples shipped with PVSnesLib for the complete source code.

