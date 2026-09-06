# Firmware

This board runs [Betaflight](https://betaflight.github.io/) (GPL-3.0), pinned here as a git submodule in `betaflight/`.

## Current version

- Submodule commit: `142ed9a` (master, 2026-08-28)

## Building

See `betaflight/` for the standard Betaflight build setup (arm-none-eabi-gcc + Makefile). The custom part for this board is a unified target config mapping:

- SPI1 = ICM-42688-P gyro/accel + BMP280 barometer
- SPI2 = AT7456E OSD
- SPI3 = microSD blackbox
- UART1/2/3/6 = pads T1-T6 / R1-R6
- motor timers, beeper, LED strip, VBAT/current ADC

## Updating the submodule

```sh
git submodule update --remote firmware/betaflight   # pull latest
```

Clone with submodules:

```sh
git clone --recursive <this repo>
```