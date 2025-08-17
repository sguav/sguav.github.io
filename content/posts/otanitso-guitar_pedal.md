+++
title = "Otanitso - A STM32 based short memory audio phrase mangler"
date = "2025-08-08"
tags = ["hw", "fw", "stm32", "audio"]
+++

`Otanitso` is the reversed of the word _ostinato_.
In music jargon it refers to a short phrase that is persistently repeated within a music piece.

In this project, I am creating a simple guitar pedal that continuously records a contained time slice of incoming audio, and when triggered replays it **reversed**.

## Prototype

I've begun developing on a spare STM32 nucleo board I had around, not really thinking of what the final product will use.

The target device was `STM32F767`, and I shunned using CubeMX/IDE to try being as bare-metal as possible.

Starting with the [STF32F76xxx Reference Manual](https://www.st.com/resource/en/reference_manual/rm0410-stm32f76xxx-and-stm32f77xxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf) I've written the linker script with the reference to system memory.

<img src="/images/stm32f76_mmap.png" alt="STM32F76xxx Memory Map" style="max-width: 100%; height: auto;">

```ld
MEMORY
{
  FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 2M
  SRAM2 (rwx): ORIGIN = 0x2007C000, LENGTH = 16K
  SRAM1 (rwx): ORIGIN = 0x20020000, LENGTH = 386K
  DTCM  (rwx): ORIGIN = 0x20000000, LENGTH = 128K
}
```

A LOT of memory.
I suppose we'll start wide and see how to pack everyting later in a smaller device.

