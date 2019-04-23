# Keebs QMK Factory

My keyboards and thus those targeted :

-  OLKB Preonic
-  OLKB Planck
-  Massdrop ALT
-  UT47.2
-  Miuni32
-  Daisy

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

```shell
docker image build --rm --pull --compress -f Dockerfile.preonic -t qmk/builder:latest .
```

```shell
docker container run --rm -mount type=bind,src="$(pwd)"/build,dst=/build/qmk/.build qmk/builder:latest
```
