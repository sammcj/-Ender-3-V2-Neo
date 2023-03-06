# Ender-3-V2-Neo

My Documentation, Configuration and Scripts for the Ender 3 V2 Neo 3d Printer.

- [Ender-3-V2-Neo](#ender-3-v2-neo)
- [Sam's Ender 3 V2 Neo Setup](#sams-ender-3-v2-neo-setup)
  - [Hardware](#hardware)
    - [Upgrades](#upgrades)
    - [Connectivity](#connectivity)
  - [Firmware](#firmware)
    - [Custom Firmware Further Customisations](#custom-firmware-further-customisations)
    - [Formatting SD Cards](#formatting-sd-cards)
  - [Settings](#settings)
    - [Feedrate calibration](#feedrate-calibration)
      - [Stock Ender 3 v2 Neo Extruder](#stock-ender-3-v2-neo-extruder)
      - [E3D Titan Extruder](#e3d-titan-extruder)
      - [LDO Orbiter 2.0 Extruder](#ldo-orbiter-20-extruder)
  - [Slicing](#slicing)
  - [Prusa Slicer](#prusa-slicer)
    - [Cura](#cura)
  - [Materials](#materials)
  - [GCode scripts](#gcode-scripts)
  - [Octoprint](#octoprint)
    - [Octoprint Docker](#octoprint-docker)
  - [Failed print recovery](#failed-print-recovery)
    - [Webcam](#webcam)
  - [Links](#links)
  - [Author](#author)
  - [Images](#images)

# Sam's Ender 3 V2 Neo Setup

## Hardware

- Ender 3 v2 Neo

### Upgrades

- Hotend is a Creality Spider v3 (300)
- E3D/Creality Titan Extruder
- Drive drive conversation
- Dual Z Axis
- Upgraded hotend fan to a 4020
- Hero Me Gen 7 (Work In Progress - Not yet in use!)
  - 2x 5015 Blower fans for part cooling
  - 1x 4020 Fan for the hotend
  - 1x 80mm Fan for the PSU

### Connectivity

- Home server (running Fedora Server)
- OctoPrint running in a container
- Webcam running with ustreamer
- Printer connected via USB
  - I did try removing the USB 5v pin as per <https://community.octoprint.org/t/put-tape-on-the-5v-pin-why-and-how/13574> but it caused the printer to not be detected, I ended up getting an adapter that removes the 5v pin cleanly.
  - udev rules for consistent USB device naming - when you reconnect a USB device it might get a different device name (e.g. `/dev/ttyUSB0` > `/dev/ttyUSB1` etc...) - this is annoying if you've configured your software such as Octoprint to use a specific device.

As such you can create udev rules to ensure both the printer and any related webcams are always named consistently (e.g. `/dev/3dprinter`, `/dev/webcam` etc...).

- See [octoprint/`99-3d-printer.rules`](octoprint/99-3d-printer.rules)

## Firmware

I have settled on using @mriscoc's Firmware for the Ender 3 V2.

It is based on the Marlin firmware but has a number of optimisations for the Ender 3 V2 Neo and is well suited to Octoprint.

- [mriscoc/Ender3V2S1](https://github.com/mriscoc/Ender3V2S1)
- [How to install the firmware](https://github.com/mriscoc/Ender3V2S1/wiki/How-to-install-the-firmware)

Note: To store the mesh and other settings on the printer you must leave a SD plugged in, otherwise when you restart the printer it will return to the firmware defaults.

### Custom Firmware Further Customisations

- See [firmware](firmware/README.md)

### Formatting SD Cards

Make sure you format your SD card with 4096 byte sector size or it may not flash correctly.

Format a SDCard with 4096 byte sector size in macOS

```shell
diskutil list # Find the SDCard you want to format

diskutil info disk4s1 # Get the current SDCard info, assuming the disk is disk4s1
sudo newfs_msdos -F 32 -b 4096 disk4s1 # Format the SDCard as FAT32 with a 4096 byte sector size, assuming the disk is disk4s1
```

## Settings

- Bed size (printable)
  - X: 220mm
  - Y: 220mm
- Bed size (max)
  - X: 245mm
  - Y: 230mm
- Feedrate (calibrated at the extruder (aka cold calibration): 95.7mm/s

### Feedrate calibration

#### Stock Ender 3 v2 Neo Extruder

(95.5+96.2+96.4)/3=96.033 = Extrusion multiplier: 1.04

Current estep: 93 = E step multiplier found before: 1.04

New estep value: 96.7, (repeat)

- Final estep value: 95.7

#### E3D Titan Extruder

Creality Titan Kit

- Final estep value: 406.0

#### LDO Orbiter 2.0 Extruder

Default config:

- E350 E16 ; micro stepping to 16
- M92 E690 ; set extruder steps/mm to 690 defaults (needs to be calibrated)
- M201 E3000 ; set max acceleration to 3000/s2 (can go to 8000, but 3000 should be good)
- M203 E120 ; set max speed to 120mm/s
- M205 E5 ; set acceleration to 5mm/s2
- M906 TO E850 ; set motor current to 850mA
- M900 TO K0.22 L0.02 ; set Linear Advance defaults (needs to be calibrated)
- M207 S1.5 F7200 Z0.2 ; firmware retraction (needs to be calibrated - 1.0~1.5mm)
- Pressure Advance: 0.2~0.3s (needs to be calibrated)
- Max. instantaneous speed change (jerk): 10mm/s (Marlin) (recommended 300/5)
- Motor current: 1.2A Peak or 0.85A RMS (LDO-36STH20-1004AHG)
- Drive motor with stealth chop disabled
- Resistor shunt for Ender 3 v2 Neo Trinamic TMC2130, TMC2208, TMC2209, TMC2225, TMC2226 - Rshunt = 0.11Ω => Vref =1.2V, Vref = Imot*(Rsunt+20mΩ)/92.85mV
  - Vref -> reference voltage measured on the trimmer resistor
  - Imot - desired peak motor current - for the Orbiter v2.0 recommended value is 1.2A
  - Rsunt - shunt resistor or sense resistor value defined in Ohms.
  - See <https://orbiterprojects.com/orbiter-v2-0/>

And remember to do a M500 to save settings

```gcode
M350 E16                        ;micro stepping set to 16*
M92 E690                        ;steps/mm - you may need to fine tune it
M201 E3000                    ;acceleration mm/s2
M203 E120                      ;max speed mm/s
M205 E5                          ;E jerk mm/s
M906 T0 E850                 ;motor RMS current in mA*
M900 T0 K0.22 L0.02      ;linear advance values to be calibrated*
M207 S1.5 F7200 Z0.2    ;firmware retraction*
M500                                ;save settings to EEPROM
```

- Calibrated estep: #TODO:

## Slicing

## Prusa Slicer

- Running the [latest beta/alpha version](https://github.com/prusa3d/PrusaSlicer/releases).
- See [prusa-slicer](prusa-slicer).

Output Options

```shell
/Users/samm/bin/ArcWelder --g90-influences-extruder;
```

### Cura

- [Ultimaker Cura](https://ultimaker.com/software/ultimaker-cura).
- See [cura/profiles](cura/profiles)

_Note: Creality's own '[Creality Slicer](https://www.creality.com/pages/download-ender-3-v2-neo)' software is a fork of [Cura](https://ultimaker.com/software/ultimaker-cura) and is often lagging behind Cura versions._

## Materials

- See [cura/materials](cura/materials)

- PLA+ (eSun, yxpolyer)
  - Extruder Temperature: 208°C
    - Initial Layer Temperature: 210°C
  - Platform Temperature: 68°C
    - Initial Layer Temperature: 75°C

## GCode scripts

- See [gcodes](gcodes)

## Octoprint

### Octoprint Docker

- See [octoprint/docker-compose.yml](octoprint/docker-compose.yml)

## Failed print recovery

I've created a GCODE template that can be used to recover/resume from a failed print.

- See [gcodes/manual-print-recovery.gcode](gcodes/manual-print-recovery.gcode)

### Webcam

Octoprint Webcam Settings

- Stream URL: `/webcam/?action=stream`
- Snapshot URL: `http://<host-ip>:9876/?action=snapshot`
- Path to FFMPEG: `/usr/bin/ffmpeg`

URLs for reaching the camera from outside the container are:

- Stream: `http://<your-server-ip>:7126/webcam/?action=stream`
- Snapshot: `http://<your-server-ip>:7126/webcam/?action=snapshot`

## Links

- <https://github.com/mriscoc/Ender3V2S1>
- <https://github.com/Lash-L/Ender-3-V2-Neo-Setup>
- <https://github.com/norpchen/ProcessGCode>

## Author

Sam McLeod

[smcleod.net](https://smcleod.net)

## Images

v4.2.2 (and v4.2.7) motherboard diagram (upscaled using ML Super Resolution)

![](images/V422andV427moboupscaled.jpg)
