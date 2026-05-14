All SNES rom have an internal header that is used to identifying the game, producer, region and technical aspects of the ROM. It's often referred as Internal ROM Header or SNES Software Specification.

Although it's not required to run a game on real hardware, the SNES ROM Header was used during the Nintendo approval process for validation and it's also used by the SNES emulators to identify and determine the memory layout and ROM type.

The data starts at SNES address $00:FFB0 and ends at $00:FFDF. $00:FFE0 though $00:FFFF contains the SNES vector information and it's actually used by the SNES CPU to determine where to execute when an interrupt occurs.

The header is located at the CPU address range $00FFC0-$00FFDF, right before the interrupt vectors, with an optional second header at $00FFB0-$00FFBF.  

This means that the location of the header within the actual ROM file will change based on the cartridge's memory map mode - with LoROM games placing it at $007Fxx, HiROM games placing it at $00FFxx, and ExHiROM games placing it at $40FFxx.  

Therefore, if it's correctly filled out, an emulator will have a higher chance of being able to figure out where the header is. See: Header Verification below.

## Cartridge header  

|ADDRESS|LENGTH|DATA NAME|  
|-------|------|---------|  
|$00:FFC0|21 bytes|Game Title Registration|  
|$00:FFD5|1 byte|Map Mode|  
|$00:FFD6|1 byte|Cartridge Type|  
|$00:FFD7|1 byte|ROM Size|  
|$00:FFD8|1 byte|RAM Size|  
|$00:FFD9|1 byte|Destination Code|  
|$00:FFDA|1 byte|Fixed Value|  
|$00:FFDB|1 byte|Mask ROM Version|  
|$00:FFDC|2 bytes|Complement Check|  
|$00:FFDE|2 bytes|Check Sum|  

## The `.SNESHEADER` Structure  

In PVSnesLib, the header is typically defined in `hdr.asm` assembly file or a library configuration file. Here is the standard structure: 

```assembly
.SNESHEADER
  ID "SNES"                      ; 4-byte ID (Always "SNES")
  
  NAME "MY EPIC GAME        "    ; Game Title Specification (21 bytes)
  
  ; FASTROM ($30) or SLOWROM ($20) + Addressing Mode
  ; LoROM = $20, HiROM = $21
  ROMBANKSIZE $8000              ; 32KB per bank (Standard LoROM)
  ROMBANKS 8                     ; Total number of 32KB banks (e.g., 2Mbits)

  SLOWROM
  LOROM

  CARTRIDGETYPE $00              ; $00 = ROM only, $01 = ROM + RAM, $02 = ROM + SRAM
  ROMSIZE $08                    ; $08 = 2 Mbits, $09 = 4 Mbits, etc.
  SRAMSIZE $00                   ; $00 = No SRAM, $01 = 16kbits, $03 = 64kbits
  COUNTRY $01                    ; $01 = USA (NTSC), $00 = Japan (NTSC), $02 = Europe (PAL)
  LICENSEECODE $00               ; Developer ID
  VERSION $00                    ; Game Version 1.0, standard for new homebrew projects
.ENDME

```
### Game Title Specification  
The game title is 21 bytes long, encoded with the JIS X 0201 character set (which consists of standard ASCII plus katakana). If the title is shorter than 21 bytes, then the remainder should be padded with spaces (0x20).  

### Map Mode

This byte defines two things: the memory mapping (LoROM vs. HiROM) and the access speed. It is mapped through keywords in header.

#### ROM Layout (`LOROM` / `HIROM`)

* **LoROM ($20):** Banks are 32KB. The upper 32KB of each 64KB bank is mapped to the SNES bus. This is the most common format for PVSnesLib projects.
* **HiROM ($21):** Banks are 64KB. This allows for larger continuous memory blocks but is more complex to map.

#### ROM Speed (`SLOWROM` / `FASTROM`)

* **SLOWROM ($20 added to layout):** Runs at 2.68 MHz. Standard for older or budget titles.
* **FASTROM ($30 added to layout):** Runs at 3.58 MHz. Requires setting a specific register in your code (`$420D`) to enable the speed boost.

| Value | Description | Common Games |
| --- | --- | --- |
| `$20` | 2.68MHz LoROM | *Super Mario World* |
| `$21` | 2.68MHz HiROM | *The Legend of Zelda: ALTTP* |
| `$30` | 3.58MHz LoROM (FastROM) | *F-Zero* |
| `$31` | 3.58MHz HiROM (FastROM) | *Chrono Trigger* |

### Cartridge Type  

Defines the hardware inside the cartridge, such as Save RAM or Coprocessors.

| Value | Hardware Configuration | Description |
| --- | --- | --- |
| `$00` | **ROM** | Basic game with no extra hardware. |
| `$01` | **ROM + RAM** | Includes extra Work RAM, but no save battery. |
| `$02` | **ROM + RAM + Battery** | Standard configuration for games with Save files. |
| `$03` | **ROM + DSP** | Includes a Fixed-point DSP chip (e.g., *Super Mario Kart*). |
| `$04` | **ROM + DSP + RAM** | DSP chip with additional buffer RAM. |
| `$05` | **ROM + DSP + RAM + Battery** | DSP chip with save capability. |
| `$13` | **ROM + Super FX** | Includes the GSU-1/2 (Super FX) for 3D math (e.g., *Star Fox*). |
| `$14` | **ROM + Super FX + RAM** | Super FX chip with extra memory. |
| `$15` | **ROM + Super FX + RAM + Battery** | Super FX chip with save capability (e.g., *Yoshi's Island*). |
| `$25` | **ROM + OBC1 + RAM + Battery** | Used specifically for *Metal Combat: Falcon's Revenge*. |
| `$34` | **ROM + SA-1 + RAM** | High-speed co-processor (e.g., *Kirby Super Star*). |
| `$35` | **ROM + SA-1 + RAM + Battery** | SA-1 with save support (e.g., *Super Mario RPG*). |
| `$43` | **ROM + S-DD1** | Graphics decompression chip (e.g., *Street Fighter Alpha 2*). |
| `$45` | **ROM + S-DD1 + RAM + Battery** | S-DD1 with save support (e.g., *Star Ocean*). |
| `$55` | **ROM + S-RTC + RAM + Battery** | Includes a Real-Time Clock (e.g., *Daikaijuu Monogatari II*). |
| `$E3` | **ROM + Other Chip** | Used for miscellaneous chips like the SRTC. |


### ROM and RAM Size

The `ROMSIZE` formula is generally $2^n$ KB.

| Value | Size | Example Game |
| --- | --- | --- |
| `$07` | 128 KB | *Combat Ribbons* |
| `$08` | 256 KB (2 Mbit) | *Super Mario Bros. (All-Stars version)* |
| `$09` | 512 KB (4 Mbit) | *F-Zero* |
| `$0A` | 1024 KB (8 Mbit) | *Super Castlevania IV* |
| `$0B` | 2048 KB (16 Mbit) | *The Legend of Zelda: A Link to the Past* |
| `$0C` | 4096 KB (32 Mbit) | *Super Metroid* |
| `$0D` | 8192 KB (64 Mbit) | *Tales of Phantasia* (Max official size) |

`SRAMSIZE` refers to the "Save RAM" (SRAM) used for battery-backed saves or extra workspace.

| Value | Size | Example Game |
| --- | --- | --- |
| `$00` | None | *Street Fighter II* |
| `$01` | 2 KB | *Super Mario World* |
| `$03` | 8 KB | *The Legend of Zelda: ALTTP* |
| `$05` | 32 KB | *Secret of Mana* |
| `$07` | 128 KB | *SimCity* |

Exception: If you're using Super FX aka GSU-1, move this value to the Expansion RAM Size field, and put #$00 in this byte.  

### License Code
Where the game is intended to be sold. Common values: #$00 - Japan; #$01 - USA; #$02 - Europe (enables 50fps PAL mode)  
Store the code, from the table below, which best describes where the product will be sold.

|Value|Destination|ROM ROM Recognition Code (Fourth digit of Game Code)|  
|-------|------|---------|  
| `$00` |Japan|J|
| `$01` |North America (USA and Canada)|E|
| `$02` |All of Europe|P|
| `$03` |Scandinavia|W|
| `$06` |Europe (French only)|F|
| `$07` |Dutch|H|
| `$08` |Spanish|S|
| `$09` |German|D|
| `$OA` |Italian|I|
| `$OB` |Chinese|C|
| `$OD` |Korean|K|
| `$OE` |Common|A|
| `$OF` |Canada|N|
| `$10` |Brazil|B|
|| `$Nintendo Gateway System|G|
| `$11` |Australia|U|
| `$12` |Other Variation|X|
| `$13` |Other Variation|Y|
| `$14` |Other Variation|Z|

### Version

This entry is primarily used for **software versioning**. When a game was re-released to fix bugs, change regional content, or update copyright information (such as Player's Choice editions or "v1.1" releases), the version byte was incremented.

* **Initial Release:** Almost every game starts with a version value of `$00`.
* **Revisions:** If a bug fix was issued, the value becomes `$01`, then `$02`, and so on.

| Value | Version | Usage |
| --- | --- | --- |
| `$00` | 1.0 | Initial release of the game. |
| `$01` | 1.1 | First revision (Bug fixes, censorship changes). |
| `$02` | 1.2 | Second revision (Rare, usually for major overhauls). |
| `$03+` | 1.3+ | Subsequent revisions. |

### Real-World Examples

#### **Example 1: Super Mario World**

*Super Mario World* is a classic LoROM game with a battery for saves.

```assembly
.SNESHEADER
  NAME "SUPER MARIOWORLD     " 
  SLOWROM
  LOROM
  CARTRIDGETYPE $02              ; ROM + RAM + Battery
  ROMSIZE $09                    ; 4 Mbit (512 KB)
  SRAMSIZE $01                   ; 2 KB SRAM
  COUNTRY $01                    ; USA
.ENDSNES

```

#### **Example 2: Chrono Trigger**

*Chrono Trigger* uses HiROM and FastROM to handle its large world and fast data access.

```assembly
.SNESHEADER
  NAME "CHRONO TRIGGER       "
  FASTROM                        ; 3.58MHz
  HIROM                          ; High ROM mapping
  CARTRIDGETYPE $02              ; ROM + RAM + Battery
  ROMSIZE $0C                    ; 32 Mbit (4 MB)
  SRAMSIZE $03                   ; 8 KB SRAM
  COUNTRY $01                    ; USA
.ENDSNES

```

### Expanded cartridge header    

|ADDRESS|LENGTH|DATA NAME|  
|-------|------|---------|  
| `$00:FFB0` |2 bytes|Maker Code|  
| `$00:FFB2` |4 bytes|Game Code|  
| `$00:FFB6` |7 bytes|Reserved, should be zero|  
| `$00:FFBC` |1 byte|Expansion flash size: 1 << N kilobytes|  
| `$00:FFBD` |1 byte|Expansion RAM size: 1 << N kilobytes - for GSU?|  
| `$00:FFBE` |1 byte|Special Version (usually 0)|  
| `$00:FFBF` |1 byte|Chipset subtype, used if chipset is $F0-$FF|  

#### Maker Code  
Two alphanumeric ASCII bytes identifying your company. The 2-digit ASCII code are assigned by Nintendo. Refer to the Nintendo/ Licensee contract, if in doubt. All letters must be in upper case.  

#### Game Code  
Four alphanumeric ASCII bytes identifying your game. The 4-digit Game Code assigned by Nintendo in ASCII. All letters must be in upper case.  
Exception: If the game code starts with Z and ends with J, it's a BS-X flash cartridge.  
