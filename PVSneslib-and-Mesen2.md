

As described on [github](https://github.com/SourMesen/Mesen2), _Mesen is a multi-system emulator (NES, SNES, Game Boy, Game Boy Advance, PC Engine, SMS/Game Gear, WonderSwan) for Windows, Linux and macOS._

<img width="1086" height="889" alt="mesen2" src="https://github.com/user-attachments/assets/7509a175-d99b-48c9-bd46-77ef45122323" />


It is the tool we recommend to use with PVSneslib on any operating system to test your developments and debug them.

## Download

First, please [download](https://github.com/SourMesen/Mesen2/releases) the version corresponding to your system then install it and validate that the emulator works correctly.

## Tutorials 1 : Visualize headroom for logic via REG_RDNMI and WAI   

### Objective  
Show the end of logic as a point in the Mesen Event Viewer and then wait for NMI. The gap between this point and NMI shows remaining CPU time before VBlank.  

Custom asm WaitForVBlank Method:  
```
WFVBlank:
  LDA.l $004210        ; read REG_RDNMI produces point in Event Viewer
  WAI                             ; wait for NMI
  rtl
  .ends
```

### Behavior  
LDA reads REG_RDNMI. Mesen displays this read as a point in the Event Viewer. WAI stops the CPU until NMI occurs. RTL returns to the caller afterward.  

### Requirement  
Your NMI handler must read or clear REG_RDNMI. If not cleared WAI may return immediately which removes the wait and invalidates the measurement.  

Gameloop pattern:

```
while(1) {
  do_logic();
  WFVBlank();
  do_vblank_dma();
}
```

### Interpretation in Mesen  
The REG_RDNMI point marks the end of logic. The NMI event marks the start of VBlank. The gap between both events is headroom. If there is no gap logic is at the limit. If NMI occurs while logic is still running frame time is exceeded.  

### Interpretation example  
The screenshot shows that only a few scanlines remain before VBlank. This indicates that an overrun is close. Consider optimizing the logic or spreading work across multiple frames. If the number of free scanlines continues to shrink a real overrun will occur.  
<img width="1024" height="1024" alt="mesen_tip" src="https://github.com/user-attachments/assets/6d2b4bae-7fd0-407a-aa6e-8c7f12f68f46" />
