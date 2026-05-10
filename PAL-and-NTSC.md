# PAL vs NTSC: What Is the Difference?

The SNES was released in different regions with hardware tuned to match the local television standard:

| Feature | NTSC (Japan / North America) | PAL (Europe / Australia) |
|---|---|---|
| Console | Super Famicom / Super NES US | Super Nintendo (EUR/AUS) |
| Video standard | NTSC | PAL |
| Vertical refresh rate | **60 Hz** | **50 Hz** |
| Visible scanlines | 224 | 224 (inside a 256-line frame) |
| Total scanlines / frame | 262 | 312 |
| Master clock | ~21.477 MHz | ~21.281 MHz |
| CPU clock (slow) | ~1.79 MHz | ~1.78 MHz |
| CPU clock (fast / FastROM) | ~3.58 MHz | ~3.55 MHz |
| SPC700 (audio CPU) | ~1.024 MHz | ~1.024 MHz |

## Why does 50 Hz matter for games?

Many games written for NTSC assume that one VBlank happens every 1/60 of a second. On a PAL console running at 50 Hz there are only 50 VBlanks per second, making the game run **~17% slower** — sprites move more slowly, music plays at a lower pitch, and timers run long. This is the infamous "PAL slowdown" players experienced with imported cartridges in Europe.

Conversely, an unmodified PAL game running at 60 Hz on an NTSC console may run faster than intended and, more importantly, may **crash** because the 50 Hz timing code does not have enough time to finish inside a 60 Hz frame.

# Impact on SNES Development

When developing a homebrew, you must decide:

- **Target only one region** — simpler, but limits your audience.
- **Support both regions** — requires runtime detection and timing adjustments.

## Frame timing

Your main game loop is usually driven by the VBlank NMI. On NTSC you have ~16.7 ms per frame; on PAL you have ~20 ms. Any per-frame logic (physics steps, timers, animation counters) must account for this difference.

## Music and SFX

The SPC700 audio CPU runs at the same frequency on both systems, so audio sample rates are identical. However, if you drive music tempo by counting NMI frames, tempo will differ between regions. Use an SPC-internal timer for tempo whenever possible.

## Raster effects and HDMA

HDMA tables are based on scanline counts. Both PAL and NTSC show 224 active lines in normal mode, so most HDMA effects transfer without change. Be careful with effects that reference the total scanline count or that run past line 224.

# Setting the Region in Your ROM Header

The SNES ROM header contains a **Destination Code** byte at address `$00:FFD9`. This byte tells the console (and emulators) which region the ROM targets, and it is how a PAL console decides to run at 50 Hz.

Common values used in homebrews:

| Value | Region | Frequency |
|---|---|---|
| `$00` | Japan | 60 Hz (NTSC) |
| `$01` | North America | 60 Hz (NTSC) |
| `$02` | Europe | 50 Hz (PAL) |
| `$11` | Australia | 50 Hz (PAL) |

In PVSnesLib the header is defined in your project's `.asm` source file via the `CARTRIDGE_INFO` block. For example:

```asm
;----------------------------------------------------------------
; ROM Header
;----------------------------------------------------------------
    .SNESHEADER
    ID      "SNES"              ; 4-char game code
    NAME    "MY HOMEBREW       " ; 21 chars, pad with spaces
    SLOWROM                     ; or FASTROM
    LOROM                       ; or HIROM
    ROMSIZE $08                 ; 256 KB = 2^8 KB
    SRAMSIZE $00                ; no SRAM
    COUNTRY $01                 ; $01 = USA/NTSC  |  $02 = Europe/PAL
    LICENSEECODE $00
    VERSION $00
    .ENDSNES
```

> For a ROM you want to work on both regions, set `COUNTRY $01` (NTSC) and handle PAL at runtime (see below). This is the safest default because NTSC timing is tighter — if your code fits inside 16.7 ms it will also fit inside the longer 20 ms PAL frame.

# Detecting PAL or NTSC at Runtime

The SNES PPU exposes the current video mode through register `$213F` (STAT78). Bit 4 of this register indicates the current field — but the reliable way to detect PAL vs NTSC is to read the **master clock** indirectly by counting scanlines per frame.

PVSnesLib exposes two global variables for this purpose:

```c
    snes_50hz  // 1 if on a PAL/50Hz SNES  

    snes_fps   // 50 if PAL console (50 Hz) or 60 if NTSC console (60Hz)  
```

PVSNesLib also exposes a function to check if the console region is identical to the one put in the cartride:  
```c
    if (consoleRegionIsOK() == 1) {
         // code for same region
    } else {
        // code if region is not the same
    }
}
```

# Switching Your Console with FFVIMan

If you develop on a PAL console (common in Europe) but want to test at 60 Hz, or vice-versa, you need to physically modify the console — or use a **switchless mod**.

[FFVIMan's Switch SNES](https://www.ffviman.fr/switch-snes/) is the most comprehensive French-language reference for SNES region modifications. The site covers the full range of mods:

# Testing on Real Hardware with a Flash Cartridge

Emulators like Mesen2 are excellent for development, but real hardware testing is essential to catch timing issues, audio differences, and PPU edge cases that emulators may not reproduce perfectly.

You can find an example of flash cartridge for SNES homebrew testing here **[Super EverDrive X5](https://krikzz.com/our-products/cartridges/spedx5.html)** by Krikzz.

