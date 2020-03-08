# Keebs QMK Factory

My keyboards and thus those targeted :

-  OLKB Preonic (MCU: ARM STM32F303) [>> see documentation](./doc/preonic/)
-  OLKB Planck (MCU: TBD) [>> see documentation](./doc/planck/)
-  Massdrop ALT (MCU: TBD) [>> see documentation](./doc/alt/)
-  UT47.2 (MCU: TBD) [>> see documentation](./doc/ut472/)
-  Miuni32 (MCU: atmega32u4) [>> see documentation](./doc/miuni32/)
-  Daisy (MCU: atmega32u4) [>> see documentation](./doc/daisy/)
-  Lubrigante Wonderland (MCU: TBD)

## Requirements

Many OS'es are supported, here we assume that Debian systems are used.

```
build-essential
avr-libc
binutils-arm-none-eabi
binutils-avr
dfu-programmer
dfu-util
diffutils
gcc
gcc-arm-none-eabi
gcc-avr
git
libnewlib-arm-none-eabi
unzip
wget
zip
```

See [qmk_firmware/util/linux_install.sh](./qmk_firmware/util/linux_install.sh)

## Build

### On Host

```shell
make planck/rev6:default
make planck/rev6:myKeymap
```

### Docker

#### Enable BuildKit

```shell
export DOCKER_BUILDKIT=1
```

#### Build Base Image

```shell
docker image build --rm --pull --compress -f Dockerfile.preonic -t qmk/builder:latest .
```

#### Compile and Flash

```shell
docker container run \
  --rm \
  --mount type=bind,src="$(pwd)"/build,dst=/build/qmk/.build \
  --privileged
  qmk/builder:latest
```

## Flash Firmware

The firware files are always named like `<keyboard_name>_<keymap_name>.{bin,hex}`

### Requirements

Setup udev to access bootloaders (DFU, HID...) without root privileges:

> /etc/udev/rules.d/49-teensy.rules

```
ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04[789B]?", ENV{ID_MM_DEVICE_IGNORE}="1"
ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04[789A]?", ENV{MTP_NO_PROBE}="1"
SUBSYSTEMS=="usb", ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04[789ABCD]?", MODE:="0666"
KERNEL=="ttyACM*", ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04[789B]?", MODE:="0666"
```

> /etc/udev/rules.d/50-atmel-dfu.rules:

```
# Atmel ATMega32U4
SUBSYSTEMS=="usb", ATTRS{idVendor}=="03eb", ATTRS{idProduct}=="2ff4", MODE:="0666"
# Atmel USBKEY AT90USB1287
SUBSYSTEMS=="usb", ATTRS{idVendor}=="03eb", ATTRS{idProduct}=="2ffb", MODE:="0666"
# Atmel ATMega32U2
SUBSYSTEMS=="usb", ATTRS{idVendor}=="03eb", ATTRS{idProduct}=="2ff0", MODE:="0666"
```

> /etc/udev/rules.d/52-tmk-keyboard.rules:

```
# tmk keyboard products     https://github.com/tmk/tmk_keyboard
SUBSYSTEMS=="usb", ATTRS{idVendor}=="feed", MODE:="0666"
```

> /etc/udev/rules.d/54-input-club-keyboard.rules:

```
# Input Club keyboard bootloader
SUBSYSTEMS=="usb", ATTRS{idVendor}=="1c11", MODE:="0666"
```

### Atmega32u4 MCU

```shell
$ dfu-tool list
Found 03eb:2ff4 [v0.0]:
 Protocol:      AtmelAVR
 Name:          ATm32U4DFU
 Serial:        1.0.0
 Mode:          DFU
 Status:        OK
 State:         dfuIDLE
 Transfer Size: 64 bytes
 Attributes:    can-download|can-upload
 Quirks:        force-dfu-mode|use-any-interface|legacy-protocol
 Chip ID:       0x581e9587
 ID:            0
  Name:         Flash
  Region 0x00:  Zone:0, Sec#:0, Addr:0x00000000, Size:0x7000, Caps:0x7 [REW]
```

#### Manually

```shell
$ dfu-programmer atmega32u4 erase
$ dfu-programmer atmega32u4 flash miuni32_default.hex
$ dfu-programmer atmega32u4 start
```

### STM32 MCU

Check that keyboard is in DFU mode, compile and flash the firmware:
```shell
$ dfu-tool --list
Found DFU: [0483:df11] ver=2200, devnum=8, cfg=1, intf=0, path="1-6", alt=1, name="@Option Bytes  /0x1FFFF800/01*016 e", serial="206B30532035"
Found DFU: [0483:df11] ver=2200, devnum=8, cfg=1, intf=0, path="1-6", alt=0, name="@Internal Flash  /0x08000000/128*0002Kg", serial="206B30532035"
```

#### Manually

Burn manually the firmware to MCU:
```shell
$ dfu-util -v -d 0483:df11 -a 0 -s 0x08000000:leave -D preonic_rev3_grumpycat.bin
```

#### With Make

Burn firmware automatically, with make:
```shell
$ TIME_DELAY=20 make preonic/rev3:default:dfu-util-wait
```

Output:

```shell
Linking: .build/preonic_rev3_default.elf                                                            [OK]
Creating binary load file for flashing: .build/preonic_rev3_default.bin                             [OK]
Creating load file for flashing: .build/preonic_rev3_default.hex                                    [OK]

Size after:
   text	   data	    bss	    dec	    hex	filename
      0	  49328	      0	  49328	   c0b0	.build/preonic_rev3_default.hex

Copying preonic_rev3_default.bin to qmk_firmware folder                                             [OK]
Preparing to flash firmware. Please enter bootloader now...
Flashing preonic_rev3_default.bin
dfu-util 0.9

dfu-util: Invalid DFU suffix signature
dfu-util: A valid DFU suffix will be required in a future dfu-util release!!!
Opening DFU capable USB device...
ID 0483:df11
Run-time device DFU version 011a
Claiming USB DFU Interface...
Setting Alternate Setting #0 ...
Determining device status: state = dfuIDLE, status = 0
dfuIDLE, continuing
DFU mode device DFU version 011a
Device returned transfer size 2048
DfuSe interface name: "Internal Flash  "
Downloading to address = 0x08000000, size = 49336
Download	[=========================] 100%        49336 bytes
Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```
