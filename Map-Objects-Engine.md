The goal is to build a simple side-scrolling platformer using:

- tile maps
- animated sprites
- the PVSnesLib map engine
- the PVSnesLib object engine

The tutorial uses a simplified level inspired by *Commander Keen 1*.

The map used in the tutorial is this one:
<a target="_blank" href="https://user-images.githubusercontent.com/2528347/198873690-096676c1-71af-4082-93ca-a2cf0a7a04c1.png"><img src="https://user-images.githubusercontent.com/2528347/198873690-096676c1-71af-4082-93ca-a2cf0a7a04c1.png"></a>

Click on the map and save it on your hard drive, inside a directory for this tutorial.

# Install Tiled

Tiled is used to create maps and place objects.

Go to https://www.mapeditor.org/ and download Tiled for your operating system (download is available at https://thorbjorn.itch.io/tiled).  

Use **Tiled 1.9.x** for compatibility with `tmx2snes`.

# Preparing the Map

## Convert a Bitmap into a TMX Map

Go to https://portabledev.com/pvsneslib/tilesetextractor/ and upload the png file your saved from Commander Keen 1 game with the "Choose File" button on the top left of the screen.

The tool will create tileset and tmx files, as below.

<img width="300" src="https://user-images.githubusercontent.com/2528347/198873692-5e1ee6d1-5ca0-4c11-a40c-196acb4853b4.png">

Save the 2 files shown below in same directory where you saved the PNG map file of the game.

<img width="300" src="https://user-images.githubusercontent.com/2528347/198873694-4672ac17-2b43-4978-8770-ad2c188ac272.png">

## Prepare Tileset Graphics

Your graphics must:

- use 256 colors
- not contain alpha transparency
- use palette index `0` as transparent color

GraphicGale can be used to edit indexed graphics:

<img width="300" src="https://user-images.githubusercontent.com/2528347/198880302-c959856b-8f34-4a8c-be0e-e6e062249249.png">

Your can also go to https://portabledev.com/pvsneslib/tiledpalettequant/ and upload the png file your saved from Commander Keen 1 game with the "Choose File" button on the top left of the screen.  

Put `1` in **palettes** textbox, `15` in **Colors per palette**, click on radio button **transparent color" to have a color `0` with pink color.  

Click **Quantize** to generate the bitmap with your options. The tool generates a bmp file you will have to convert to png.

Use your graphic editor software to change color depth of tiles to 256 colors without alpha channel (or gfx4snes will not work...). 
In our example, the number of colors will certainly be OK, we're just validating that the extractor set the number of colors correctly.

<img width="300" src="https://user-images.githubusercontent.com/2528347/198880302-c959856b-8f34-4a8c-be0e-e6e062249249.png">

Store both files in your project directory.

# Configure the Map in Tiled

Open `tiled.tmx` with Tiled, it will open Tiled with your png file converted to a map file compatible with PVSnesLib!

<img width="400" src="https://user-images.githubusercontent.com/2528347/198880481-b873e585-757f-4f8b-99e9-ed06cfab5bd7.png">

> The BG layer name `BG1` is important. This layer name will later be used by the engine.  

## Tile Properties

On the screen below, click on "Edit Tileset" button to open a new tab with tileset properties.

<img width="300" src="https://user-images.githubusercontent.com/2528347/199168309-7d0eabf0-f314-48e4-b199-b2d89a7c927c.png">

If you used the converter tool described in previos chapter, you will have 3 properties for each tile ("attribute", "palette" and "priority"). If not, select all the tiles on the right with the mouse and use the "+" button on the bottom left to add the 3 properties.

Each tile should contain these properties:

| Property | Purpose |
|---|---|
| attribute | collision / gameplay |
| palette | palette selection |
| priority | sprite priority |

## Collision Attributes

Now select the tiles as shown below (red rectangles to show the tiles) to change their "attribute" property to FF00 to describe them as blocker. Our hero will not be able to pass through them.

<img width="500" src="https://user-images.githubusercontent.com/2528347/199170180-98bdf0f9-992a-44cf-aa08-a8c2a0be923e.png">

Do the same with pillars (again, red rectangles to show the tiles) to change priority property to 1, to allow our hero to pass behind them.

<img width="500" src="https://user-images.githubusercontent.com/2528347/199170185-6f9115a9-4e0d-4580-a16a-06947b0abf03.png">

If you have a tileset with multiple palettes, you can do the same with the "palette" property of each tile.

The property `attribute` is special, some values are managed by the map/object engines.
* **FF00** is for solid tiles, objects can't pass through them
* **0002** will change action property of object to ACT_BURN value (see <a href="https://github.com/alekmaul/pvsneslib/blob/master/pvsneslib/include/snes/object.h">object.h</a> file of PVSneslib)
* **0004**  will change action property of object to ACT_DIE value (see <a href="https://github.com/alekmaul/pvsneslib/blob/master/pvsneslib/include/snes/object.h">object.h</a> file of PVSneslib)

_`attribute` property value **0002** or **0004** will need to be managed in your code._

## Export map in jSON format

To export your map in a format usable with PVSnesLib, you need to click on the first tab named **tiled.tmx**, to be on the map file and not on the tiles part of the map. You also need to use menu **Map**, option **Map properties...** to have the correct properties.
Select **CSV** as **Tile Leyer Format**.

<img width="300" src="https://user-images.githubusercontent.com/2528347/199171995-261bc3d5-94c2-4404-999a-d3d889b4df09.png">

Then, choose **File/Export** and save the file in json format. Name the file **tiled.tmj** (Type is **JSON map files(*.tmj *.json)**). Next time, Tiled will not ask you about a filename as you previously saved your file in json format.

<img width="300" src="https://user-images.githubusercontent.com/2528347/199171998-588929f3-19c8-4027-a36e-c6725e0ab25e.png">

## Adding Objects

Objects are entities managed by the PVSnesLib object engine.

Each object has:

- an initialization function
- an update function
- optionally a render function

## Adding objects with Tiled

Dans Tiled, nous nous limiterons à définir l’emplacement des objets. Il faut juste prendre soin de créer le premier objet en tant que personnage principale pour notre jeu, les autres seront les objets avec lesquels notre personnage pourra avoir des interactions.

Cette définition se fait au travers du calque nommé « Entities ». Cela n’est pas obligatoire et peut être fait directement dans le code. C’est juste par simplicité et facilité de mise à jour que nous utilisons Tiled dans notre cas.


## Object Classes

| Class | Object |
|---|---|
| 0 | Hero |
| 1 | Monster |

## Example Objects

### Hero

| Property | Value |
|---|---|
| Class | 0 |
| Position | 168,208 |

### Monster

| Property | Value |
|---|---|
| Class | 1 |
| Position | 520,224 |
| minx | minimum X movement |
| maxx | maximum X movement |

The monster uses custom properties:

```text
minx
maxx
```

These define movement boundaries.


# Create Project Structure

Here is the project layout we will use:

```text
project/
├── Makefile
├── hdr.asm
├── data.asm
├── displaymap.c
├── hero.c
├── monster.c
├── tiled.tmj
├── tiles.png
├── sprkeen.png
└── sprmonster.png
```

First of all, we will convert all assets to data compatible with PVSnesLib.

## Convert Tiles graphics

```make
tiles.pic: tiles.png
	@echo convert map tiles ... $(notdir $@)
	$(GFXCONV) -s 8 -o 48 -u 16 -p -m -i $<
```

## Convert Map

```make
BG1.m16: tiled.tmj tiles.pic
	@echo convert map tiled ... $(notdir $@)
	$(TMXCONV) $< tiled.map
```

## Convert Hero Sprite

```make
sprkeen.pic: sprkeen.png
	@echo convert hero sprite bitmap ... $(notdir $@)
	$(GFXCONV) -s 16 -o 16 -u 16 -p -i $<
```

## Convert Monster Sprite

```make
sprmonster.pic: sprmonster.png
	@echo convert monster sprite bitmap ... $(notdir $@)
	$(GFXCONV) -s 16 -o 16 -u 16 -p -i $<
```

## Build Dependencies

```make
bitmaps : BG1.m16 sprkeen.pic sprmonster.pic tiles.pic

all: bitmaps $(ROMNAME).sfc
```

## Add resources

The `data.asm` file contains binary resources included in the ROM.

```asm
.include "hdr.asm"

.section ".rodata1" superfree
.include "tiles_data.as"

mapkeen: .incbin "BG1.m16"
tilesetatt: .incbin "tiled.b16"
tilesetdef: .incbin "tiled.t16"
objmap: .incbin "tiled.o16"
.ends

.section ".rodata2" superfree
.include "sprkeen_data.as"
.include "sprmonster_data.as"
.ends
```


# Main Program Initialization

The main program initializes:

- background layers
- sprite engine
- object engine
- map engine

## Initialize Backgrounds

```c
bgInitTileSet(
    0,
    &tiles_til,
    &tiles_pal,
    0,
    (&tiles_tilend - &tiles_til),
    16 * 2,
    BG_16COLORS,
    0x2000
);

bgSetMapPtr(0, 0x6800, SC_64x32);
```

`0x6800` is required by the map engine.

---

# Initialize Dynamic Sprites

```c
oamInitDynamicSprite(0x0000, 0x1000, 0, 0, OBJ_SIZE8_L16);
```

- `0x0000` : large sprites VRAM area
- `0x1000` : small sprites VRAM area

---

# Initialize Object Engine

```c
objInitEngine();

objInitFunctions(0, &heroinit, &heroupdate, NULL);
objInitFunctions(1, &monsterinit, &monsterupdate, NULL);
```

---

# Load Map and Objects

```c
objLoadObjects((char *)&objmap);

mapLoad(
    (u8 *)&mapkeen,
    (u8 *)&tilesetdef,
    (u8 *)&tilesetatt
);
```

Object initialization functions are automatically called during map loading.

---

# Configure Video Mode

```c
setMode(BG_MODE1, 0);

bgSetDisable(1);
bgSetDisable(2);

setScreenOn();
```

---

# Main Game Loop

```c
while (1)
{
    pad0 = padsCurrent(0);

    mapUpdate();

    objUpdateAll();

    oamInitDynamicSpriteEndFrame();

    WaitForVBlank();

    mapVblank();

    oamVramQueueUpdate();
}
```

This loop:

1. reads controller input
2. updates the map
3. updates objects
4. uploads sprite graphics
5. synchronizes with VBlank

---

# Hero Object

## Hero Variables

```c
t_objs *heroobj;

s16 *heroox, *herooy;
s16 *heroxv, *heroyv;

u16 herox, heroy;

u8 herofidx, flip;
```

---

# Hero Initialization

```c
void heroinit(u16 xp, u16 yp, u16 type, u16 minx, u16 maxx)
```

The object is allocated using:

```c
objNew(type, xp, yp)
```

Configure dimensions:

```c
heroobj->width = 16;
heroobj->height = 24;
heroobj->yofs = 8;
```

---

# Hero Sprite Setup

The hero uses two `16x16` sprites.

```c
oambuffer[0].oamframeid = 0;
oambuffer[0].oamrefresh = 1;
oambuffer[0].oamattribute = 0x20 | (0 << 1);
oambuffer[0].oamgraphics = &sprkeen_til;
```

Load the palette:

```c
setPalette(&sprkeen_pal, 128 + 0 * 16, 16 * 2);
```

---

# Hero Movement

The update function handles:

- left/right movement
- jumping
- acceleration
- animation
- collisions

## Move Left

```c
*heroxv -= HERO_ACCEL;
```

## Move Right

```c
*heroxv += HERO_ACCEL;
```

## Jump

```c
*heroyv = -(HERO_JUMPING);
```

High jump:

```c
*heroyv = -(HERO_HIJUMPING);
```

---

# Map Collision

```c
objCollidMap(idx);
```

This checks collision against tile attributes configured in Tiled.

---

# Updating Object Position

```c
objUpdateXY(idx);
```

---

# Rendering the Hero

```c
oamDynamic16Draw(0);
oamDynamic16Draw(1);
```

The camera follows the hero:

```c
mapUpdateCamera(herox, heroy);
```

---

# Hero Animations

Animations are controlled through:

- `ACT_WALK`
- `ACT_FALL`
- `ACT_JUMP`
- `ACT_STAND`

Walking animation updates sprite frame indices.

Jumping switches to dedicated jump frames.

---

# Monster Object

The monster behaves similarly to the hero but uses simpler logic.

## Initialization

```c
monsterobj->width = 16;
monsterobj->height = 16;

monsterobj->xmin = minx;
monsterobj->xmax = maxx;
```

Movement limits come from Tiled custom properties.

---

# Monster Sprite Index

```c
monsterobj->sprnum = 2;
```

Sprites `0` and `1` are already used by the hero.

---

# Monster Movement

The monster automatically moves left and right:

```c
if (monsterx <= monsterobj->xmin)
```

and:

```c
if (monsterx >= monsterobj->xmax)
```

The sprite is flipped depending on direction.

---

