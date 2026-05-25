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

In Tiled, we will limit ourselves to defining the location of objects. This definition is done through the layer named “Entities”. 

<img width="609" height="432" alt="15_image" src="https://github.com/user-attachments/assets/ce69b40c-9a60-46a3-841a-6d2a59fbe879" />

> This is not required and can be done directly in code. It’s just for simplicity and ease of updating that we use Tiled in our case.

We just have to take care of creating the first object as the main character for our game, the others will be the objects with which our character can have interactions.  

<img width="233" height="114" alt="16_image" src="https://github.com/user-attachments/assets/bc462e81-afc7-4c66-82b7-bd41f2b46036" />

Object types are defined by the `Class` attribute of Tiled. The `Class` with value 0 will therefore be the one used for our character. We will also add another class 1 object which will be a watch in our game in next chapters.  

<img width="678" height="468" alt="17_image" src="https://github.com/user-attachments/assets/42e7d937-60d3-4049-af80-f981f3756c51" />

We thus add the Hero object of class `0` in coordinates **168.208** and the Monster object of class `1` in coordinates **520.224**.

<img width="853" height="261" alt="18_image" src="https://github.com/user-attachments/assets/fd8de242-78c0-4641-b67a-d6cb15f1b152" />

One particular thing concerns the monster, we manage 2 custom properties (like for the tile attributes) named `minx` and `maxx`, which will allow us to update the movement of the monster more easily on the map. These 2 properties contain the minimum and maximum values ​​in X of its movements. There is no equivalent to manage the same thing in Y.

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

## Convert Hero and Mosnter Sprites

```make
sprkeen.pic: sprkeen.png
	@echo convert hero sprite bitmap ... $(notdir $@)
	$(GFXCONV) -s 16 -o 16 -u 16 -p -i $<

sprmonster.pic: sprmonster.png
	@echo convert monster sprite bitmap ... $(notdir $@)
	$(GFXCONV) -s 16 -o 16 -u 16 -p -i $<
```

Hero and Monster sprites are defined in our graphic tool, `GraphicGale' in our case.

<img width="598" height="535" alt="18_1_image" src="https://github.com/user-attachments/assets/97fc4adc-3aa7-46bc-91dc-41a23e5fe1ae" />

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

## Initialize Dynamic Sprites

```c
oamInitDynamicSprite(0x0000, 0x1000, 0, 0, OBJ_SIZE8_L16);
```

- `0x0000` : large sprites VRAM area
- `0x1000` : small sprites VRAM area

## Initialize Object Engine

```c
objInitEngine();

objInitFunctions(0, &heroinit, &heroupdate, NULL);
objInitFunctions(1, &monsterinit, &monsterupdate, NULL);
```

## Load Map and Objects

```c
objLoadObjects((char *)&objmap);

mapLoad(
    (u8 *)&mapkeen,
    (u8 *)&tilesetdef,
    (u8 *)&tilesetatt
);
```

Object initialization functions are automatically called during map loading.

## Configure Video Mode

```c
setMode(BG_MODE1, 0);

bgSetDisable(1);
bgSetDisable(2);

setScreenOn();
```

## Main Game Loop

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
6. update map scrolling and free sprite queue

## Hero Object implementation

Hero source code uses some variables for its management and an object pointer to allow us to manage easily each proprety of the hero.

```c
t_objs *heroobj;

s16 *heroox, *herooy;
s16 *heroxv, *heroyv;

u16 herox, heroy;

u8 herofidx, flip;
```

### Hero Initialization

```c
void heroinit(u16 xp, u16 yp, u16 type, u16 minx, u16 maxx)
{
    // Prepare new object
    if (objNew(type, xp, yp) == 0)
        // no more space, we quit
        return;

    // Init. sprite object (objgetid is id of current object)
    //  like sprite size (16x24 with an offset Y deof 8, see sprite graphic)
    objGetPointer(objgetid);
    heroobj = &objbuffers[objptr - 1];
    heroobj->width = 16; heroobj->height = 24; heroobj->yofs=8;
    
    // Save velocity and coordinate pointers
    heroox = (u16 *)&(heroobj->xpos + 1);
    herooy = (u16 *)&(heroobj->ypos + 1);
    heroxv = (short *)&(heroobj->xvel);
    heroyv = (short *)&(heroobj->yvel);

    // Init other variables
    herofidx = 0;
    heroobj->action = ACT_STAND;
```

The object is allocated using `objNew(type, xp, yp)`

The hero uses two `16x16` sprites as its size is `16x24`.

```c
    // prepare le sprite (2 sprites de 16x16)
    oambuffer[0].oamframeid = 0;
    oambuffer[0].oamrefresh = 1;
    oambuffer[0].oamattribute = 0x20 | (0 << 1); // palette 0 des sprites and sprite 16x16 and priorite 2 
    oambuffer[0].oamgraphics = &sprkeen_til;
    oambuffer[1].oamframeid = 1;
    oambuffer[1].oamrefresh = 1;
    oambuffer[1].oamattribute = 0x20 | (0 << 1); // palette 0 des sprites and sprite 16x16 and priorite 2 
    oambuffer[1].oamgraphics = &sprkeen_til;

    // Init palette du sprites 
    setPalette(&sprkeen_pal, 128 + 0 * 16, 16 * 2);
```

### Hero Movement

The update function handles:

- left/right movement
- jumping
- acceleration
- animation
- collisions


```c
void heroupdate(u8 idx)
{
    // check only the keys for the game
    if (pad0 & (KEY_RIGHT | KEY_LEFT | KEY_A))
    {
        // go to the left
        if (pad0 & KEY_LEFT)
        {
            // update anim (sprites 2-3)
            oambuffer[0].oamattribute |= 0x40; // flip sprite
            oambuffer[1].oamattribute |= 0x40; // flip sprite

            // update velocity
            heroobj->action = ACT_WALK;
            *heroxv -= (HERO_ACCEL);
            if (*heroxv <= (-HERO_MAXACCEL))
                *heroxv = (-HERO_MAXACCEL);
        }
        // go to the right
        if (pad0 & KEY_RIGHT)
        {
            // update anim (sprites 2-3)
            oambuffer[0].oamattribute &= ~0x40; // don't flip sprite
            oambuffer[1].oamattribute &= ~0x40; // don't flip sprite

            // update velocity
            heroobj->action = ACT_WALK;
            *heroxv += (HERO_ACCEL);
            if (*heroxv >= (HERO_MAXACCEL))
                *heroxv = (HERO_MAXACCEL);
        }
        // jump 
        if (pad0 & KEY_A)
        {
            // we can jump only if we are on ground
            if ((heroobj->tilestand != 0))
            {
                heroobj->action = ACT_JUMP;
                // if key up, jump 2x more
                if (pad0 & KEY_UP)
                    *heroyv = -(HERO_HIJUMPING);
                else
                    *heroyv = -(HERO_JUMPING);
            }
        }
    }
```
### Map Collision

Inside the heroUpdate function, we check collision against tile attributes configured in Tiled, update position regarding `X` and `Y` velocity.

```c
    // 1), check les collisions avec la carte
    objCollidMap(idx);

    //  met a jour l'animation suivant l'etat du heros
    if (heroobj->action == ACT_WALK)
        herowalk(idx);
    else if (heroobj->action == ACT_FALL)
        herofall(idx);
    else if (heroobj->action == ACT_JUMP)
        herojump(idx);

    // met a jour la position sur la carte
    objUpdateXY(idx);

    // quelques limites ;)
    if (*heroox <= 0)
        *heroox = 0;
    if (*herooy <= 0)
        *herooy = 0;
```

### Rendering the Hero and camera position

```c
    // change les coordonnées du sprites sur la position sur la carte
    herox = (*heroox);
    heroy = (*herooy);
    oambuffer[0].oamx = herox - x_pos;
    oambuffer[0].oamy = heroy - y_pos;
    oambuffer[1].oamx = herox - x_pos;
    oambuffer[1].oamy = heroy - y_pos+16;
    oamDynamic16Draw(0);
    oamDynamic16Draw(1);

    // Met a jour la camera suivant la position du heros
    mapUpdateCamera(herox, heroy);
```

### Hero Animations

Animations are controlled through:

- `ACT_WALK`
- `ACT_FALL`
- `ACT_JUMP`
- `ACT_STAND`

Walking animation updates sprite frame indices. Jumping switches to dedicated jump frames.

```c
// Gestion du deplacement du heros
void herowalk(u8 idx)
{
    // update animation
    flip++;
    if ((flip & 3) == 3)
    {
        herofidx+=2;
        if (herofidx>6) herofidx = 0;
        oambuffer[0].oamframeid = herofidx;
        oambuffer[0].oamrefresh = 1;
        oambuffer[1].oamframeid = herofidx+1;
        oambuffer[1].oamrefresh = 1;
    }

    // check if we are still walking or not with the velocity properties of object
    if (*heroyv != 0)
        heroobj->action = ACT_FALL;
    else if ((*heroxv == 0) && (*heroyv == 0))
        heroobj->action = ACT_STAND;
}

//---------------------------------------------------------------------------------
void herofall(u8 idx)
{
    // Si on ne chute plus, on reste debout
    if (*heroyv == 0)
    {
        heroobj->action = ACT_STAND;
        oambuffer[0].oamframeid = 0;
        oambuffer[0].oamrefresh = 1;
        oambuffer[1].oamframeid = 1;
        oambuffer[1].oamrefresh = 1;
    }
}

//---------------------------------------------------------------------------------
void herojump(u8 idx)
{
    // change sprite
    if (oambuffer[0].oamframeid != 8)
    {
        oambuffer[0].oamframeid = 8;
        oambuffer[0].oamrefresh = 1;
        oambuffer[1].oamframeid = 9;
        oambuffer[1].oamrefresh = 1;
    }

    // if no more jumping, then fall
    if (*heroyv >= 0)
        heroobj->action = ACT_FALL;
}

```


## Monster Object implementation

The monster behaves similarly to the hero but uses simpler logic.

```c
monsterobj->width = 16;
monsterobj->height = 16;

monsterobj->xmin = minx;
monsterobj->xmax = maxx;
```

Movement limits come from Tiled custom properties.

### Monster Sprite Index

>Pay attention to the number of sprites to use. 

Often, we manage it in a variable that we increment each time an initialization function is called. As we only have one monster object on the screen, I leave it set to the value “2” which corresponds to the sprite we are going to have (0 and 1 being for the hero).

```c
    monsterobj->sprnum = 2;  // IMPORTANT ! To be managed with a variable depending on the number of objects on the screen (2 for the hero, so there are 2 here)
```

> Sprites `0` and `1` are already used by the hero.

### Monster Movement

The update function is simpler because we only manage the movement. 

We start by retrieving the object identifier and the sprite is flipped depending on direction.

```c
void monsterupdate(u8 idx)
{
    // recupere l'objet a mettre a jour
    monsterobj = &objbuffers[idx];
    monsterox = (u16 *)&(monsterobj->xpos + 1);
    monsteroy = (u16 *)&(monsterobj->ypos + 1);
    monsternum=monsterobj->sprnum;
    monsterx = *monsterox;
```

and the limit managed with minx and maxx

```c
    monsterobj->count++;
    if (monsterobj->count >= 3) { // updates 20 times per second
        monsterobj->count = 0;
        monsterobj->sprframe = (1 - monsterobj->sprframe); // Faster because only 2 frames
        oambuffer[monsternum].oamframeid=monsterobj->sprframe; 
        oambuffer[monsternum].oamrefresh = 1;

        // We go to the left
        if (monsterobj->dir == MONSTER_LEFT) {
            if (monsterx <= monsterobj->xmin) {
                monsterobj->dir = MONSTER_RIGHT;
                monsterobj->xvel = +MONSTER_XVELOC;
                monsterobj->yvel = 0;
                oambuffer[monsternum].oamattribute &=~0x40;
            }
            else {
                monsterobj->xvel = -MONSTER_XVELOC;
            }
        }
        // here it's right :)
        else {
            if (monsterx >= monsterobj->xmax) {
                monsterobj->dir = MONSTER_LEFT;
                monsterobj->xvel = -MONSTER_XVELOC;
                monsterobj->yvel = 0;
                oambuffer[monsternum].oamattribute |=0x40;
            }
            else {
                monsterobj->xvel = +MONSTER_XVELOC;
            }
        }
        // update coordinates
        objUpdateXY(idx);
    }

    // update spite on screen
    oambuffer[monsternum].oamx = monsterx - x_pos;
    oambuffer[monsternum].oamy =(*monsteroy) - y_pos;
    oamDynamic16Draw(monsternum);
```

That's all for the map and sprite engines, you can now play with them for your own game!
