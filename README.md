# nanoBoot

<!-- not used by osamu: [![Build Status](https://travis-ci.org/volium/nanoBoot.svg?branch=master)](https://travis-ci.org/volium/nanoBoot) -->

This repository contains the source code for the USB HID-based bootloader for ATmegaXXU4 family of devices.

The name *nanoBoot* comes from the fact that the compiled source fits in the smallest available boot size on the ATMegaXXu4 devices, 256 words or 512 bytes. The code is based on Dean Camera's [LUFA](https://github.com/abcminiuser/lufa) USB implementation, but it is **EXTREMELY** streamlined, size-optimized and targeted for the [ATmega16U4](http://www.atmel.com/devices/atmega16u4.aspx) and [ATmega32u4](http://www.atmel.com/devices/atmega32u4.aspx) devices.

Initial and major portion of manual assembly code optimization efforts were performed by [volium](https://github.com/volium) and published as the original [nanoBoot](https://github.com/volium/nanoBoot).  Further tweakings were performed by osamu to allow arbitrary setting for CKDIV8 fuse and published here on `devel` branch in the forked [nanoBoot](https://github.com/osamuaoki/nanoBoot) repository to allow arbitrary CKDIV8 settings and to turn on and off LED.

There are a few hardware assumptions to the fuse settings which make this bootloader as compact as possible.

<!-- It's very likely that a few sections can be rewritten to make it even smaller, and the ultimate goal is to support EEPROM programming as well, although that would require changes to the host code. -->

<!--The current version (commit #[5401f92](https://github.com/volium/nanoBoot/commit/5f84c0c44d78e907de869176c576855c8ba7a2f2)) is supported as-is in the 'hid_bootloader_loader.py' script that ships with [LUFA-151115](https://github.com/abcminiuser/lufa/releases/tag/LUFA-151115), and is exactly 508 bytes long.-->

<!-- The current version (2020-04-03) is supported as-is in the 'hid_bootloader_loader.py' script that ships with [LUFA-151115](https://github.com/abcminiuser/lufa/releases/tag/LUFA-151115), and is exactly 512 bytes long. -->

The current version (2020-04-10) is tested manually with the compiled `hid_bootloader_cli.c` from [LUFA](https://github.com/abcminiuser/lufa) 597fbf47c ("Merge pull request #156 from jessexm/dragndrop", 2020-02-11) on Debian GNU/Linux 10 (buster).

Required packages on Debian GNU/Linux system: `gcc-avr`, `avr-libc`, `binutils-avr`, `libusb-dev`, `build-essential`, `git`

## HW assumptions:

* CLK is 16 MHz Crystal and fuses are setup correctly to support it:
    * Select Clock Source (CKSEL3:CKSEL0) fuses are set to Extenal Crystal, CKSEL=1111 SUT=11
    * Divide clock by 8 fuse (CKDIV8) can be any value.
* Bootloader starts on reset; Hardware Boot Enable fuse is configured, HWBE=0
* Boot Flash Size is set correctly to 256 words (512 bytes), StartAddress=0x3F00, BOOTSZ=11
* Device signature = 0x1E9587

* Fuse Settings:
    * lfuse memory = 0xFF or 0x7F (CKDIV8=1 or 0, 16CK+65ms)
    * hfuse memory = 0xD6 (EESAVE=0, BOOTRST=0)
    * efuse memory = 0xC7 (=0xF7, No BOD)

* Alternatively BOD can be used to ease CKSEL-SUT setting requirements to
  allow teensy like FUSE setting
    * lfuse memory = 0x5F (CKDIV8=0, 16CK + 0ms)
    * hfuse memory = 0xDF (EESAVE=1, BOOTRST=1)
    * efuse memory = 0xC4 (=0xF4, BOD=2.4V)

* LED on D6 port (Configurable in #define)

## Usage

Please install this bootloader `nanoBoot.hex` using the ISP connected programmer.

```
$ sudo avrdude -v -p atmega32u4 -c avrisp2 -Pusb -e -U flash:w:nanoBoot.hex \
             -U lfuse:w:0x5f:m -U hfuse:w:0xdf:m -U efuse:w:0xc4:m
```

You can start this bootloader by connecting the board to the PC with USB cable and pressing the RESET button.  It is good idea to monitor the PC's USB connection.

```
 $ watch lsusb
```

If this bootloader is started, you should see "Atmel".

Please note, now this bootloader turns on LED just before sending device ID.  Thus monitoring of USB is now optional.

(If LED doesn't turn on even after 10 second wait for any reason, press the RESET button again.)

Then program MCU with, e.g., a `LED.hex` firmware as:

```
 $ sudo hid_bootloader_cli -mmcu=atmega32u4 -v LED.hex
```
Please note, this bootloader turns off LED upon finish programming.

(Pressing the RESET button during active bootloader execution seems to halt the bootloader.  This seems to be the reason you need to press the RESET button again.)

## Documentation

The documentation is part of the source code itself, and even though some people may find it extremely verbose, I think that's better than lack of documentation; after all, assembly can be hard to read sometimes... ohhh yes, in case that was not expected, this is all written in pure GAS (GNU Assembly), compiled using the [Atmel AVR 8-bit Toolchain](http://www.atmel.com/tools/atmelavrtoolchainforwindows.aspx).
