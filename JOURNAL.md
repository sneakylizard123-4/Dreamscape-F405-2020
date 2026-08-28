---
title: Dreamscape F405 devlog
author: sneakylizard123-4
description: Building a custom STM32F405 flight controller from scratch
created_at: 2026-08-02
---

# August 2: Picking a brain (and the parts around it)

Decided to finally build a 20x20 Flight controller. FPV boards are basically all STM32 + a gyro, and the STM32F405 felt like the right middle ground - plenty of SPI/UARTs, USB, cheap, and it's what runs all the firmware I know (Betaflight). Read the F405 datasheet more carefully than I've ever read one. Also lined up the big pieces: ICM-42688-P for the gyro/accel, BMP280 barometer, AT7456E OSD chip, a microSD slot for blackbox logs. Ironed out the top-level plan of what goes on which SPI bus so the sheets would stay sane.

![the plan](images/schematic/Dreamscape-F405-2020.png)

**Total time spent: 3 hours**

# August 5: Power sheet

First sheet drawn: power. I don't want to feed the rail from a battery and nothing else, so there are two LMR51430 bucks out of the battery - one making 5 V, one making 10 V for the camera/VTX. The 10 V one is gated by a firmware pin (`10V_EN`) so I'm not powering the VTX when I don't need to. Then a TLV75733 LDO drops the 5 V to 3.3 V for the MCU and sensors. The part that took the longest was the TPS2116 power mux between the 5 V buck and USB - I wanted the board to auto-switch to USB power when plugged in and not fight the USB port. Plus a divider for reading battery voltage into the ADC.

![power sheet](images/schematic/Dreamscape-F405-2020-Power.png)

**Total time spent: 3 hours**

# August 8: Sensors sheet

Gyro + barometer. ICM-42688-P on SPI1 with one chip-select, BMP280 on the same SPI bus with its own CS, plus INT1/INT2 pulled out to the MCU for interrupts. I stuck to the reference circuits in the datasheets pretty hard here - the gyro is the whole reason a board like this works, so I didn't want to fudge it. Wiring the two interrupts over to free pins on the F405 took a little while since I wanted to keep the INT pins scalable if I migrate.

![sensors sheet](images/schematic/Dreamscape-F405-2020-Sensors.png)

**Total time spent: 2 hours**

# August 11: Blackbox sheet

microSD on SPI3. Went with a push-pull slot (TF-021B-H265) rather than a micro holder so it sits low on the board. There's a handful of pull-ups and the usual power filtering. Nothing glamorous, but I've seen enough "blackbox card not found" dramas on forums to know I should actually read the SD spec notes instead of assuming it'll just work.

![blackbox sheet](images/schematic/Dreamscape-F405-2020-Blackbox.png)

**Total time spent: 2 hours**

# August 14: OSD sheet

AT7456E OSD - the classic analog overlay chip. It needs an external 27 MHz crystal, so that got added with its load caps. The video path goes CAM in -> OSD -> VTX out, which means the chip has to sit in the middle of the video signal and I had to be careful that the signal path makes sense on the sheet and later on the board. This is one of those chips where the datasheet layout matters, so I mostly followed it.

![osd sheet](images/schematic/Dreamscape-F405-2020-OSD.png)

**Total time spent: 2 hours**

# August 18: STM32 sheet

The big one - drawing the F405 itself. LQFP-100 is a lot of pins to place and route mentally. Got the 8 MHz crystal + boot config, and happily this chip has USB on it, which means the board can flash over USB via the built-in DFU bootloader and I don't have to expose a debug header to get firmware on it initially. I spent most of the time just assigning pins so that the SPI buses, UARTs, I2C, ADC and the timer outputs for the motors all land on usable pins without collisions.

![stm32 sheet](images/schematic/Dreamscape-F405-2020-STM32.png)

**Total time spent: 4 hours**

# August 20: Pads sheet

All the ways in and out of the board. A JST-SH 8-pin vertical connector carrying battery plus the four motor signals plus current sensing and telemetry, and a 4-pin horizontal one for an ExpressLRS receiver. Then the edge solder pads: motor pads, UARTs 1/2/3/6, I2C, beeper, LED strip, video in/out, and battery voltage. ~35 pads total. This is the "how do you actually plug this thing in" sheet, so I kept checking it against how I'd wire a real quad.

![pads sheet](images/schematic/Dreamscape-F405-2020-Pads.png)

**Total time spent: 3 hours**

# August 22: USB-C sheet

Added a USB-C connector. The F405 had drifting toward a micro-B or a header, but USB-C is what I have cables for by the truckload now. VBUS goes into the TPS2116 mux so plugging in USB powers the whole board even without a battery, and there are the two 5.1k pull-down resistors on the CC lines to make it act as a proper USB-C device. D+/D- head to the MCU pins I'd already reserved. Small sheet but it ties the board together - flashing, power, and the ELRS tune-up port are all USB now.

![usb sheet](images/schematic/Dreamscape-F405-2020-USB.png)

**Total time spent: 1.5 hours**

# August 25: Placement and start of routing

Everything on the board: 20 x 20 mm, which is .. ambitious. Components in: F405 in the middle, buck converters along one edge, camera/video stuff kept away from the high-current switching so the OSD doesn't show a tornado. Placed the two JST plugs on opposite edges so the harnesses don't tangle. Started routing and immediately rediscovered that 20 x 20 with this many pads is mostly an exercise in "where does the ground pour go."

![board so far](images/render-top.png)

**Total time spent: 4 hours**

# August 27: Routing done (+ DRC)

Called routing done. Ran DRC a bunch and fixed the usual suspects - a couple of clearance complaints near the crystal, some silk that was too close to pads, and retraced the current sense line since it runs past a buck converter on its way to the ADC. Two-layer board, so the ground pour is doing a lot of the return-path work. Nothing left but the last cosmetic pass and hitting fab when the design doc side is finished.

![finished routing](images/render-bottom.png)

**Total time spent: 6 hours**

# August 28: Silkscreen labels on the pads

The pad silkscreen bugged me for a while. Every flight controller you buy has the pads labeled so you know what you're soldering, and mine were just bare. Given how small this board is (20 x 20 mm) I kept having to look at the schematic to double-check which pad was which. That gets annoying fast when you're squeezing wires in.

So I went through and labeled every pad on the board. To make them fit next to the pads I shortened some names: `GND` became `GN` (there are six of them), `SDA`/`SCL` became `DA`/`CL`, and everything else kept its net name - `3V3`, `5V`, `10V`, `BAT`, `BZ+`/`BZ-`, `LED`, `CAM`, `VTX`, `CC`, and the UART pads `T1`-`T6`/`R1`-`R6`. It's all in the silkscreen layer so it comes out with the solder mask when I get it fabbed.

![Fresh PCB render with the pad labels](images/render-top.png)

Honestly the pickiest part was spacing. The pads ring the whole edge of the board, and with tiny 0201 parts and the STM32 in the middle there's not a lot of room off the edges. Some labels had to be pushed in or tweaked so they didn't overlap the solder pads or the copper. I didn't route anything new for this, just text in F.SilkS, but it took a few passes to stop them colliding.

**Total time spent: 1 hour**