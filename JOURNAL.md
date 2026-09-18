---
title: Dreamscape F405 devlog
author: sneakylizard123-4
description: Building a custom STM32F405 flight controller from scratch
created_at: 2026-08-02
---

# August 2: Picking a brain (and the parts around it)

started project
using 20x20 mounting pattern
planning on a 3" fpv drone
this time the fc will have everything:
- STM32F405 MCU
- ICM-42688-P IMU
- BMP280 Barometer
- AT7456E OSD
- MicroSD Slot

![the plan](images/schematic/Dreamscape-F405-2020.png)

**Total time spent: 3 hours**

# August 5: Power sheet

First sheet: POWER!!!
Using2  LMR51430 Buck converters:
- 5v always on
- 10v with enable
Using TLV75733 for 3.3V from 5v
Using TPS2116 power mux between 5v buck and USB 5v
Board auto-switches between USB and buck
And a divider to measure battery voltage

![power sheet](images/schematic/Dreamscape-F405-2020-Power.png)

**Total time spent: 3 hours**

# August 8: Sensors sheet

started sensors sheet:
- ICM-42688-P IMU
- BMP280 Barometer
Copied reference circuits
IMU and Baro share the same SPI bus (SPI1)

![sensors sheet](images/schematic/Dreamscape-F405-2020-Sensors.png)

**Total time spent: 2 hours**

# August 11: Blackbox sheet

microSD on separate SPI bus
Using for Blackbox storage
Using push-pull slot

![blackbox sheet](images/schematic/Dreamscape-F405-2020-Blackbox.png)

**Total time spent: 2 hours**

# August 14: OSD sheet

Using AT7456E for OSD
Super common OSD chip
I have to be careful about the signal path
Camera -> OSD -> VTX

![osd sheet](images/schematic/Dreamscape-F405-2020-OSD.png)

**Total time spent: 2 hours**

# August 18: STM32 sheet

started f405 sheet
Using 8MHz crystal
This chip has USB on it
no other usb chips needed
spent lots of timing choosing which pins to assign

![stm32 sheet](images/schematic/Dreamscape-F405-2020-STM32.png)

**Total time spent: 4 hours**

# August 20: Pads sheet

Started pads sheet
JST-SH for connector to ESC
also a dedicated connector for the ELRS
Copying F405 Mini pads

![pads sheet](images/schematic/Dreamscape-F405-2020-Pads.png)

**Total time spent: 3 hours**

# August 22: USB-C sheet

Started USB-C sheet
usb vbus goes into power mux so we can use the board without a battery

![usb sheet](images/schematic/Dreamscape-F405-2020-USB.png)

**Total time spent: 1.5 hours**

# August 25: Placement and start of routing

started routing pcbs.
F405 in the center
Power along one edge
sensitive analog stuff kept away from the power stage
Using inner ground layer
hopefully 4-layer will be enough

![board so far](images/render-top.png)

**Total time spent: 4 hours**

# August 27: Routing done (+ DRC)

finished routing, ran drc
some clearance issues near crystal
some silk that was too close to pads
retraced curent sense line because it runs past the buck converters on its way to the esc
power routing difficult and so is high speed data
will fix drc later

![finished routing](images/render-bottom.png)

**Total time spent: 6 hours**

# August 28: Silkscreen labels on the pads

started pad silkscreen labels
each product fc has silkscreen on the pads
labeled every pad on the board
using abbreviated labels cos too long

![Fresh PCB render with the pad labels](images/render-top.png)

spacing is so hard
especially with the 0201 parts, i need steady hands

**Total time spent: 1 hour**