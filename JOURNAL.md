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

![the plan](images/schematic/Dreamscape-F405-2020.png)

**Total time spent: 3 hours**

# August 5: Power sheet

using buck converters to step down to 5v and 3.3v
did other power stuff

![power sheet](images/schematic/Dreamscape-F405-2020-Power.png)

**Total time spent: 3 hours**

# August 8: Sensors sheet

started sensors sheet for imu, baro, and others
using spi mostly

![sensors sheet](images/schematic/Dreamscape-F405-2020-Sensors.png)

**Total time spent: 2 hours**

# August 11: Blackbox sheet

microSD blackbox for flight stuff

![blackbox sheet](images/schematic/Dreamscape-F405-2020-Blackbox.png)

**Total time spent: 2 hours**

# August 14: OSD sheet

added osd, so i can see whats going on inside the fc too, not just camera

![osd sheet](images/schematic/Dreamscape-F405-2020-OSD.png)

**Total time spent: 2 hours**

# August 18: STM32 sheet

started f405 sheet, lots of pins needed to route

![stm32 sheet](images/schematic/Dreamscape-F405-2020-STM32.png)

**Total time spent: 4 hours**

# August 20: Pads sheet

started pads sheet
copying f405 mini pads

![pads sheet](images/schematic/Dreamscape-F405-2020-Pads.png)

**Total time spent: 3 hours**

# August 22: USB-C sheet

added usb-c sheet
for data and power, usb 5v goes to a switch

![usb sheet](images/schematic/Dreamscape-F405-2020-USB.png)

**Total time spent: 1.5 hours**

# August 25: Placement and start of routing

started routing pcbs.
hopefully 4-layer will be enough

![board so far](images/render-top.png)

**Total time spent: 4 hours**

# August 27: Routing done (+ DRC)

finished routing, ran drc
power routing difficult and so is high speed data
will fix drc later

![finished routing](images/render-bottom.png)

**Total time spent: 6 hours**

# August 28: Silkscreen labels on the pads

started pad silkscreen labels

using abbreviated labels cos too long

![Fresh PCB render with the pad labels](images/render-top.png)

spacing is so hard

**Total time spent: 1 hour**