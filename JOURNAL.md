---
title: Dreamscape F405 devlog
author: sneakylizard123-4
description: Building a custom STM32F405 flight controller from scratch
created_at: 2026-08-02
---

# August 2: Picking a brain (and the parts around it)

## What I did:

- started the project
- picked the brain: STM32F405, the same chip most modern flight controllers run
- settled on a 20x20 mounting pattern so it fits standard quad stacks
- planning a 3" fpv drone around it
- picked the rest of the part list:
    - ICM-42688-P IMU
    - BMP280 barometer
    - AT7456E OSD

## Why:

- last builds used off-the-shelf flight controllers, this one is mine end to end
- everything on one board, no stacking random modules

## Screenshots:

![the plan](images/schematic/Dreamscape-F405-2020.png)

**Total time spent: 3 hours**

# August 5: Power sheet

## What I did:

- first sheet: power
- two LMR51430 bucks: a 5v rail always on, a 10v rail gated by an enable pin
- TLV75733 LDO drops 5v to 3.3v for the logic
- TPS2116 power mux picks between the 5v buck and USB 5v
- resistor divider onto an ADC pin to read battery voltage

## Why:

- camera and VTX want 10v but only when armed, so the gate keeps them off on the bench
- the mux means the board boots off USB with no battery plugged in
- battery divider sized so 6S stays under the MCU's 3.3v

## Screenshots:

![power sheet](images/schematic/Dreamscape-F405-2020-Power.png)

**Total time spent: 3 hours**

# August 8: Sensors sheet

## What I did:

- ICM-42688-P IMU
- BMP280 barometer
- both on the same SPI1 bus, separate chip selects
- copied reference circuits for the decoupling

## Why:

- the IMU is the core of the flight controller, went with the modern part instead of the old 6xx series
- baro gives pressure hold for free, costs one more chip
- sharing one SPI bus saves pins, chip select keeps them from talking over each other

## Screenshots:

![sensors sheet](images/schematic/Dreamscape-F405-2020-Sensors.png)

**Total time spent: 2 hours**

# August 11: Blackbox sheet

## What I did:

- microSD slot on its own SPI3 bus
- push-pull slot so a card clicks in
- card select pulled up, unused data lines left in SPI mode

## Why:

- blackbox logging writes constantly, didn't want it fighting the gyro for the bus
- a stuck card read could delay the IMU and wreck flight behavior

## Screenshots:

![blackbox sheet](images/schematic/Dreamscape-F405-2020-Blackbox.png)

**Total time spent: 2 hours**

# August 14: OSD sheet

## What I did:

- AT7456E for the overlay
- 27MHz crystal it needs
- kept the signal path straight: Camera -> OSD -> VTX

## Why:

- AT7456E is everywhere in FPV, tons of reference material
- the video path is analog and easy to mess up, short and clean is the only sane option

## Screenshots:

![osd sheet](images/schematic/Dreamscape-F405-2020-OSD.png)

**Total time spent: 2 hours**

# August 18: STM32 sheet

## What I did:

- started the f405 sheet
- 8MHz crystal on PH0/PH1
- this chip has USB built in, no transceiver chip needed
- spent a long time on pin assignment, matching each pad to pins with the right alternate function

## Why:

- pin choice decides the whole board, worth doing carefully upfront instead of rerouting later
- USB means flashing over the type-C port, no extra chip

## Screenshots:

![stm32 sheet](images/schematic/Dreamscape-F405-2020-STM32.png)

**Total time spent: 4 hours**

# August 20: Pads sheet

## What I did:

- pads sheet
- JST-SH connector for the ESCs and the battery/telemetry/current harness
- separate header for the ELRS receiver
- modeled on the F405 Mini pad layout

## Why:

- JST-SH is what flight controllers actually ship with, plugs match common motor and ext boards
- separate ELRS connector keeps the radio wiring clean

## Screenshots:

![pads sheet](images/schematic/Dreamscape-F405-2020-Pads.png)

**Total time spent: 3 hours**

# August 22: USB-C sheet

## What I did:

- USB-C sheet
- VBUS goes into the power mux
- 5.1k CC pull-downs

## Why:

- standard USB-C connector is cheap and everywhere
- plugging in USB powers the board through the mux, no battery needed to flash

## Screenshots:

![usb sheet](images/schematic/Dreamscape-F405-2020-USB.png)

**Total time spent: 1.5 hours**

# August 25: Placement and start of routing

## What I did:

- started the PCB
- F405 in the center
- power stage along one edge
- analog stuff kept away from the buck converters
- inner ground plane

## Why:

- traces near the power stage carry switching noise, keeping analog away saves headaches
- the ground plane gives clean return paths

## Screenshots:

![board so far](images/render-top.png)

**Total time spent: 4 hours**

# August 27: Routing done (+ DRC)

## What I did:

- finished routing both layers
- ran DRC: clearance issues near the crystal, some silk too close to pads
- retraced the current sense line, it ran past both bucks on its way to the ESC

## Why:

- power traces carry huge ripple, the current sense pickoff needed a cleaner route
- left the cosmetic DRC leftovers for the cleanup pass

## Screenshots:

![finished routing](images/render-bottom.png)

**Total time spent: 6 hours**

# August 28: Silkscreen labels on the pads

## What I did:

- labeled every pad in silkscreen
- 5v, gn, 3v3, 10v, bat, bz, led, cam, vtx, the UART pads
- shortened names so they fit on a 20x20 board

## Why:

- every real flight controller has this, makes wiring the stack at the bench not guesswork
- 0201 parts and pad labels in one place is cramped, spacing got fiddly, expect careful soldering

## Screenshots:

![Fresh PCB render with the pad labels](images/render-top.png)

**Total time spent: 1 hour**

# August 29: Design review by a second pair of eyes

## What I did:

- Forge wants the design sanity-checked by someone else, so I sent the board out for review
- the reviewer mapped every MCU pin against the STM32F405 datasheet
- went through SPI1/2/3, the UARTs, USB, the power tree, and the motor timers
- the reviewer found a real bug: the I2C pads landed on pins that can't do I2C on the F405

## Why:

- a reviewed design is cheaper to fix than a fabbed one
- I had assumed the pads were fine, fresh eyes caught what I'd stopped looking at

## Screenshots:

![stm32 sheet under review](images/schematic/Dreamscape-F405-2020-STM32.png)

**Total time spent: 3 hours**

# August 30: Fixing the I2C wiring

## What I did:

- SDA pad was on PC8: no I2C alternate function there at all
- SCL was on PC9, which is actually the I2C3 SDA pin, so they were backwards too
- re-routed SDA to PC9 and SCL to PA8, the pins that really can do I2C3
- re-ran ERC and DRC after the change
- regenerated the schematic PNGs so the repo images match the fix

## Why:

- the chip is labeled I2C3 on those pins in the datasheet, the pad wiring had drifted from it
- catching it before fab saved a dead pair of pads in the final board

## Screenshots:

![pads sheet after fix](images/schematic/Dreamscape-F405-2020-Pads.png)

**Total time spent: 2 hours**

# August 31: Firmware drop-in and STEP export

## What I did:

- added Betaflight as a git submodule under firmware/
- noted which target config to build from: SPI1 gyro and baro, SPI2 OSD, SPI3 SD, UART pads for the radio and GPS
- exported the board as STEP for the CAD requirement

## Why:

- the board is built to run stock Betaflight, keeping it as a submodule pins the upstream version we target
- the STEP export is required for submission and doubles as a sanity check of the 3D models

## Screenshots:

![board render for the CAD pass](images/render-bottom.png)

**Total time spent: 2 hours**

# September 1: Repo cleanup and submission prep

## What I did:

- rewrote the README to the Hack Club template
- reworked the journal into the what/why/screenshots format
- added a gitignore so the kicad autosave junk stays out of the repo
- noticed the JST-SH connector has no 3D model, so it shows up missing in the STEP export

## Why:

- submission needs the repo organized: BOM, sources, STEP, firmware, folders
- a clean repo is part of the review, messy history looks unfinished

## Screenshots:

![top render for the README](images/render-top.png)

**Total time spent: 2 hours**