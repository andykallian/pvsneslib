The remaining entries here on the unofficial FAQ are maintained by the community.

# Frequently asked questions (FAQ)

**[Miscellaneous](#MiscSection)**

- [I want to contribute to this project](#i-want-to-contribute-to-this-project)
- [I want to ask something](#MiscSection_2)
- [I found a bug. What can i do ?](#MiscSection_3)
- [Can we create Hirom games with the lib ?](#MiscSection_4)
- [Where comes from PVSnesLib name ?](#MiscSection_5)
- [I would like to share my project with PVSneslib community](#MiscSection_6)
- [Is it possible to use Docker ?](#MiscSection_7)
- [How to update WLA submodule to the latest commit ?](#MiscSection_8)

**[About the lib](#AboutPVsneslib)**

- [Is it possible to rotate a picture ?](#AboutPVsneslib_1)
- [What is the function to clear text ?](#AboutPVsneslib_2)
- [How can i display special characters in text ?](#AboutPVsneslib_3)
- [How to create random number ?](#AboutPVsneslib_4)
- [What is the goal of each tool ?](#AboutPVsneslib_5)
- [What are .it files ?](#AboutPVsneslib_6)
- [How to convert .mid to .it ?](#AboutPVsneslib_7)

**[Common errors](#CommonErrorsSection)**

- [What is CHECH_HEADERS error ?](#CommonErrorsSection_1)
- [Colors of my loaded picture are wrong](#CommonErrorsSection_2)
- [I get FIX_LABELS error when i build my project](#CommonErrorsSection_3)
- [Can i load sprites in 0x0000 ?](#CommonErrorsSection_4)
- [Why HDMA channel 0 doesn't work ?](#CommonErrorsSection_5)
- [Soundbank files are missing in music samples](#CommonErrorsSection_6)
- [My music has some glitch during playing](#CommonErrorsSection_7)
- [Programmer's Notepad add text anywhere when i compile](#CommonErrorsSection_8)
- [Font system doesn't work with some background mode](#CommonErrorsSection_9)
- [How to build tcc 816 provided with PVSneslib sources ?](#CommonErrorsSection_10)
- [I get the error "echo: command not found"](#CommonErrorsSection_11)
- [On Linux i get : "fatal error: bits/libc-header-start.h"](#CommonErrorsSection_12)
- [Using malloc with PVSneslib](#CommonErrorsSection_13)

**[Maps](#MapsSection)**

- [How to create maps with 16x16 tiles ?](#MapsSection_1)
- [Backgrounds begin at x = 0 and y = 1](#MapsSection_2)
---

## <a name="MiscSection"/>Miscellaneous

### <a name="MiscSection_1"/>I want to contribute to this project

PVSneslib main developer and community will be happy! You can create a fork of this project and submit your code by pullrequest.
If you want to create or update wiki pages, you can do it directly. Wiki pages are more pleasant with pictures and illustration but github requires to give a link. To avoid to lost pictures in a future if it is store anywhere on internet, you can use [this page](https://github.com/alekmaul/pvsneslib/issues/26) especially did for it. To upload your image, just add a comment, copy/paste it in the page and copy hyperlink created.

### <a name="MiscSection_2"/>I want to ask something

There is no more official forum for PVSneslib. The best thing to do is to ask on [Snesdev](https://forums.nesdev.com/viewforum.php?f=12) forum.

### <a name="MiscSection_3"/>I found a bug. What can i do ?

Please create a minimal code to reproduce it then create an [issue](https://github.com/alekmaul/pvsneslib/issues) on github explaining the bug.

### <a name="MiscSection_4"/>Can we create Hirom games with the lib ?

No, PVSneslib currenty manage Lorom games only but maybe it will implemented in future !

### <a name="MiscSection_5"/>Where comes from PVSnesLib name ?

The library could not be named LibSnes because an other project with same name was already existing. 
The "PV" letters comes from **P**ortablde**V**, the name of Alekmaul's [website](https://www.portabledev.com).

### <a name="MiscSection_6"/>I would like to share my project with PVSneslib community

It is a very good idea and the community will be probably happy to study it. But we cannot add your code with others examples in PVsneslib because we need to keep it up to date for each improvement of PVSneslib.

In this case, we recommend you to upload it on your github page and create a wiki page here to present it.

### <a name="MiscSection_7"/>Is it possible to use Docker ?

Yes, some people worked on it and created a docker image that you can find [here](https://github.com/Crazy-Piri/pvsneslib-docker)

### <a name="MiscSection_8"/>How to update WLA submodule to the latest commit ?

You just need to execute `git submodule update --remote --merge` but please keep in mind that upgrading WLA has often impact on PVSneslib and require to be fully tested.

---

## <a name="AboutPVsneslib"/>About the lib

### <a name="AboutPVsneslib_1"/>Is it possible to rotate a picture ?

Only background in mode7 can be rotate.

### <a name="AboutPVsneslib_2"/>What is the function to clear text ?

There is no function to clear text directly, you need to draw spaces on text instead like that :
```
consoleDrawText(1,2,"some text on the screen");
// will be cleared with :
consoleDrawText(1,2,"                       ");
```

### <a name="AboutPVsneslib_3"/>How can i display special characters in text ?

PVSneslib let you manage only characters from ascii code 32 to 127. If you need other characters, you have to implement it yourself (and share it with the community !).
For more informations about text in PVSneslib, you can consult [this page](https://github.com/alekmaul/pvsneslib/wiki/Input-and-Output)

### <a name="AboutPVsneslib_4"/>How to create random number ?

If you want to create random numbers, you can use rand() function available in PVSnesLib. You do not need to initialise something, it is managed by the library.
rand() function will works only if you assigned its result to `u16` variable type.
To limit numbers between values like 0 and 50, you have to do `(rand() % 50)`. But keep in mind that modulos, multiplications and division operations are very very slow on 65C816 processor used by SNES and try to avoid using it when possible.

### <a name="AboutPVsneslib_5"/>What is the goal of each tool ?

- bin2txt : convert binary file to text file to include it directly in your project. It is used to include the file generated (with TASM tool) for spc700 CPU (audio) directly in PVSneslib source code.

- gfx2snes : transform all kinds of graphics to resources in Super Nintendo format. This is a central tool used by PVSneslib to load sprites, maps or palettes.
Remember you that we have a lot of constraints on SNES and each parameter of the tool is important ! Parameters are divided in 3 groups : **g** for Graphics, **m** for Map, **p** for Palette.

- smconv : to convert musics or sounds to bank of sounds readable on Super Nintendo and its sound processor SPC700

- snesbrr : to convert .wav files to .brr. If you want more informations on this format, you can read [this page](https://wiki.superfamicom.org/bit-rate-reduction-(brr))

- snestools : this tool is not used anymore by PVSneslib (it was used to patch the rom after its build) but it still provided with it if you want to see the header of your rom.

- 816-tcc and 816-opt.py : this is the tiny C compiler for 8/16 bits architecture, it translate your C code to ASM for 65c816. Due to some limitations and performances issues, a python script is used after to optimized produced code.

- constify : by using your .c source files to detect const variables, it moves variables from RAM to ROM in .asm files to improve performances.

- tasm : it is the assembler tool for SPC700 (the processor used to manage the sounds and musics on SNES)

- wla-65816 and wlalink : these tools are assembler to convert your .asm files to .obj files (code readable by 65c816). Wlalink is the linker that "merge" all files.

### <a name="AboutPVsneslib_6"/>What are .it files ?

.it files are impulse tracker files, a music format near of SNES one (.spc files). It is used because it managed many channels like the SNES and a tool is provided with PVSnesLib (smconv) to convert it to soundbanks.
[Schism Tracker](http://schismtracker.org/) can be used to open this files, it is a reimplementation of Impulse Tracker

### <a name="AboutPVsneslib_7"/>How to convert .mid to .it ?

[OpenMPT (Open ModPlug Tracker)](https://openmpt.org/features) is able to do it !

---

## <a name="CommonErrorsSection"/>Common errors

### <a name="CommonErrorsSection_1"/>What is CHECH_HEADERS error ?

Your game have to be build with same parameters than we have in PVSnesLib header. This is because .obj files from the library are embedded with .obj files of you project which contains its own header too. It is an improvement to do with PVSnesLib!

### <a name="CommonErrorsSection_2"/>Colors of my loaded picture are wrong

It is an issue with your color palette. We do not recommend to use MS Paint which mix color strangly. You can use [Graphics Gale](https://graphicsgale.com/us/) software if you want to edit easily your colors.
This is a sample when editing sprite with MS Paint from DynamicSprite sample:

![Palette issue](https://user-images.githubusercontent.com/48180545/56434580-727f1d00-62d5-11e9-94a1-1feb1e9d591b.png)

but your initial sprite looks like that :

![initial sprite](https://user-images.githubusercontent.com/48180545/56434581-727f1d00-62d5-11e9-8762-095da6a992b6.png)

Like you saw it in [sprite tutorial](https://github.com/alekmaul/pvsneslib/wiki/Sprites), your transparency color must be the first in your palette. If it is not the case, open Graphics gale and remove unused colors from your palette by using this menu :

![Graphics gale remove unused colors](https://user-images.githubusercontent.com/48180545/56434583-7317b380-62d5-11e9-9670-497e43cb22b2.png)

In this case, transparency color is Magenta (RGB : 255 0 255). In the same menu, use "save palette" and open it in a text editor (this .pal file is not the same that .pal file generated with gfx2snes tool).
Find the line which contains your transparency color and move it to the first line of colors. Be careful, the first lines are essential informations for the tool:

![Move transparency color](https://user-images.githubusercontent.com/48180545/56434582-7317b380-62d5-11e9-80cb-c934547af546.png)

Now, import your new palette file in Graphics gale. In the "Load Palette" window, click "All" to replace colors and save your new file with palette correctly sorted.

Do not forget to clean your project (with make clean command) to try your new sprite.

### <a name="CommonErrorsSection_3"/>I get FIX_LABELS error when i build my project

It is recommended to split your code in multiple files but do not declare global variables in .h files.
Declare the variable within your .c source file:

**commonFile.c**

```
#include "commonFile.h"

bool myCommonVariable = false;

...
```

then use extern in other places (either .c files, or an .h file that .c files include) :

**main.c**

```
#include "commonFile.h"

extern bool myCommonVariable;

...
```

and do not forget to protect your header files from multiple inclusion:

**commonFile.h**

```
#ifndef COMMON_FILE_H
#define COMMON_FILE_H

...

#endif
```

### <a name="CommonErrorsSection_4"/>Can i load sprites in 0x0000 ?

Yes, you can do it but remember that PVSneslib text management begin at 0x0000. So if you want to use font system provided with the library, you need to begin after.

### <a name="CommonErrorsSection_5"/>Why HDMA channel 0 doesn't work ?

HDMA channel 0 is used internally for all DMA functions.

### <a name="CommonErrorsSection_6"/>Soundbank files are missing in music samples

Soundbank files (sounbank.asm and soundbank.h) are automatically generated in music sample when your project is compiled. To manage the project, this files are added in project tree but not in the folder like other generated files (.pal, .map, ...).

### <a name="CommonErrorsSection_7"/>My music has some glitch during playing

This is because you didn't respect snesmod requirements! It should be very useful to implement a tool to manage it automatically if possible... Take a look of pvsneslib_snesmod.txt in pvsneslib and follow this advices from **KungFuFurby** :

> Sample loop points must be divisible by 16. Loop points not
> divisible by 16 may not convert correctly. If you don't use loop
> points divisible by 16, at least make sure the loop length is
> divisible by 16.
> I use Schism Tracker to perform the job of loop point analysis and
> loop point adjustments since I can just simply type in the numbers.

> I use a calculator to take care of loop point problems simply related
> to the sample being at the wrong sample rate to have loop point
> lengths divisible by 16 (the length of the looping portion of the
> sample should at the very least be divisible by 16). I usually perform
> this on samples with shorter loop point lengths. I don't think it
> works so well on ones with longer loop point lengths, mainly because
> by then you're probably not dealing with simple waveforms as loops.

> Using the Bass & Lead sample as an example...

> Loop point is currently defined as...
> Start: 3213
> End: 3382

> That's a loop length of 169.

> I like using powers of 2 for my loop points so that if I have to
> decrease the quality of the sample, then I can do so as painlessly as
> possible to the sample (unless I find the degraded quality to be a bad
> idea), so that means 128 is the value I use here.

> Divide 169 by 128 gets you an unusual decimal number... copy that number.

> Now get the full length of the sample (that's 3383 for this sample)
> and divide by that decimal number you acquired earlier (169/128).
> You'll most likely get another unusual decimal number. Round that off
> and there's your new length that you will resize the sample to.

> I use Alt-E in Schism Tracker to perform the resize sample command.
> The program will ask for the new size of the sample.

> Now you should have a loop length that is divisible by 16. You can
> perfect the loop point by adjusting them so that the loop point
> themselves are divisible by 16.

> Only one sample can be defined per instrument...
> You'd have to duplicate the instruments and then enter the sample ID
> for all of those notes... and then you have to redefine the instrument
> IDs depending on the pitch from the old note table. Yeah...

> I thought in one case, I saw the pitch go too high (it went over
> 128khz). That's because I noticed a hiccup in the notation.
> For the one song that has this problem, I usually resize the sample
> and make it half of its length... however, I may have to make
> additional adjustments depending on how the loop points are holding up
> (length may or may not be involved, although usually I'm checking the
> loop points themselves to make sure that they are divisible by 32 or
> some higher power of 16... this indicates how many times you can cut
> the sample in half).

> Note Cut is the only New Note Action supported for SNESMod.
> One of these songs is the most visibly affected by this problem, and
> that's because SNESMod doesn't virtually allocate channels. You have
> to modify the patterns so that the note off commands go where the note
> would originally play, and the new note is put on another channel.

### <a name="CommonErrorsSection_8"/>Programmer's Notepad add text anywhere when i compile

You will note it when your project will give strange error during compilation. This is because some pieces of code are paste anywhere. In this case, save your project and close Programmer's Notepad. Open the file that don't compile **with an other tool** (like a simple Notepad) to remove bad lines and save it. Then you can continue with Programmer's Notepad software.

### <a name="CommonErrorsSection_9"/>Font system doesn't work with some background mode

The output system is only available for **BG_MODE1**. If you need it in other mode, you need to develop it.


### <a name="CommonErrorsSection_10"/>How to build tcc 816 provided with PVSneslib sources ?

Go to tcc-65816 directory, then execute **./configure** command to create the config.mak file.
If you are on windows and get an error like "'.' is not recognized as an internal or external command", you probably need to execute the **sh** command before.
After this command, you can build tcc by doing : **make 816-tcc.exe**


### <a name="CommonErrorsSection_11"/>I get the error "echo: command not found"

You probably have an issue with the format of your PVSNESLIB_HOME environment variable.
The value must be in unix style (**/c/snesdev** instead of **c:\\snesdev**) to avoid this issue. The variable can be created with this command line : `setx PVSNESLIB_HOME "/c/snesdev"`


### <a name="CommonErrorsSection_12"/>On Linux i get : "fatal error: bits/libc-header-start.h"

When building some tools on Linux like **snestools**, if you get the error _/usr/include/stdlib.h:25:10: fatal error: bits/libc-header-start.h: no such file or directory_, it is related to the -m32 CFLAG provided in the makefile, you probably forgot to install some libraries from gcc. For example on Ubuntu, you just have to install **gcc-multilib** by executing `sudo apt-get install gcc-multilib`

### <a name="CommonErrorsSection_13"/>Using malloc with PVSneslib

I want to use malloc in my program, so I am trying something like this: u16 *myHudBuffer = (u16*)malloc(160*sizeof(u16)); but it is not working. malloc requires stdlib.h which I think does not work with pvsneslib.

You can't use malloc with PVSneslib, you need to declare an array with a fixed size, u16 myHudBuffer[160] in this case.


## <a name="MapsSection"/>Maps

### <a name="MapsSection_1"/>How to create maps with 16x16 tiles ?

It is not possible to load maps with 16x16 tiles with the existing version of gfx2snes. You need to use 8x8 tiles or update the tool !
The "-gs" parameter (for "graphic size") in gfx2snes is only applied to sprites.


### <a name="MapsSection_2"/>Backgrounds begin at x = 0 and y = 1

It is not a bug and it is linked to a technical constraint :

> Note that many games will set their vertical scroll values to -1 rather than 0. This is because the SNES loads OBJ data for each scanline during the previous scanline. The very first line, though, wouldn't have any OBJ data loaded! So the SNES doesn't actually output scanline 0, although it does everything to render it.

If you want more informations on it, you can consult [this page.](https://wiki.superfamicom.org/backgrounds#toc-3)