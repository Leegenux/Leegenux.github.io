---
title: PIC32MX Basics --- Basic Configuration
date: 2019-07-06 10:59:25
thumbnail: http://pub9j4x8t.bkt.clouddn.com/pic32mx_bt_audio_pim.jpg
tags: [embeded, PIC32, PIC32MX]
---

Before we start, there are something you should know about the tutorial.

This series of tutorial will show you how to get PIC32 functioning based on a **PIC32MX220F032B** chip and a **PICKit 3** programmer. The tutorial mostly applies to Microchip PIC32MX1xx/2xx series because they are alike, and not that many differences exist between PIC32MX1xx/2xx and other chip version.

This post will discuss the basic configurations of the PIC32MX220F032B including watchdog, sysclk, secondary oscillator, JTAG, ICSP and other.

The device pinout is provided here as reference:

<img src="http://pub9j4x8t.bkt.clouddn.com/PIC32MXPinout.jpg"/>

## Option of Reference

Basically, what I am gonna talk about are based on the official help document:  [PIC32 Configuration Settings](http://www.mrc.uidaho.edu/mrc/people/jff/340/341/ECE341_Reference_Materials/Microchip%20XC32%20C%20Compiler%20and%20Libraries/PIC32ConfigSet.pdf)



## Config Bits

The PIC32 configuration bits are a set of non-volatile options defining some certain behaviors of the MCU. They are different from the program written in C or assembly in that they are stored in boot flash memory rather than program flash memory.

You should use #pragma directive to define the config bits.



### Watchdog Timer

A watchdog timer (WDT) is a hardware timer that automatically resets the computer system if the main controller neglects to periodically clear its bit.

It's widely adopted by complex embedded device for temporarily recovering from hardware fault or not responding. However, it's kind of a hassle to small scale projects sometimes.

```
#pragma config FWDTEN = off    // Disable the watchdog timer
```



### System Clock

There are two PIC32 internal oscillators. For PIC32MX220F032B, one is the 8MHz FRC (Fast Resistor Capacitor) oscillator and the other is the 3.15 MHz LPRC (Low Power Resistor Capacitor) oscillator. 

The FRC and external oscillators can be frequency scaled by in-circuit [PLL](https://en.wikipedia.org/wiki/Phase-locked_loop). For the sake of special structure of PLL, the input frequency must be between certain range. To my PIC32MX220F032B, the range is constrained to 4-5MHz. Also for each MCU, there is a recommended maximum frequency for the CPU in it. So, there are frequency dividers before and after the PLL.

Look at the example below for getting the maximum frequency of 40MHz with internal oscillator:

```
#pragma config FNOSC = FRCPLL   // Use fast resistor capacitor oscillator
#pragma config FPLLIDIV = DIV_2 // PLL input divider
#pragma config FPLLMUL = MUL_20 // PLL multipler
#pragma config FPLLODIV = DIV_2 // PLL output divider
```



### Secondary Oscillator

The Secondary Oscillator (SOSC) is intended for low-power operation with an external 32 to 100 KHz oscillator. When enabled, the external crystal should be connected to the SOSCO and SOSCI device pins.

The code below demonstrates how to enable the secondary oscillator and set it as system clock:

```
#pragma config FSOSCEN = ON  // Enable secondary oscillator 
#pragma config FNOSC = SOSC  // Make the secondary oscillator the system clock
```

In tutorials later, we adopt the internal oscillator solution.



### JTAG

JTAG (Joint Test Action Group) is one of the common standards designed for a debugging and programming interface. Since we are using PICKit3 instead, we should properly disable JTAG. JTAG occupies certain pins when enabled, and if you know a bit about the inner implementation of PPS (Peripheral Pin Selections), you will come to the conclusion that the data of the JTAG pins are two-way.

TMS, TDO, TCK and TDI are four pins required by JTAG standard. To disable it and free these pins for other I/O usages, put code below in your source.

```
#pragma config JTAGEN = OFF
```



### ICSP (In-Circuit Serial Programming)

The PGEC (clock) and PGED (data) respectively transmit programming clock and data. There are 3 pairs of PGEX pins on **PIC32MX220F032B**. If you want PGEC2 and PGED2, use the following config bits:

```
#pragma config ICESEL = ICS PGx2
```



## Configuration Optimization

The MCU can adjust its behavior according to the SYSCLK for best optimized performance. In  addition to the complex manual code modification approach, the hardware vendor has already provided us a useful tool to achieve this ------ SYSTEMConfigPerformance(int sysclk);

It takes as input argument the system clock frequency, and then enables data and instruction caches , optimizes the access time to the flash memory and caches. So always include invoke of this particular function.

For example:

```
#define SYSCLK 40000000L
int main(void){
	SYSTEMConfigPerformance(SYSCLK);
	// Code your application below
}
```

