# Preonic

## Compile Firmware

-  QMK
QMK support can be found [here](https://github.com/qmk/qmk_firmware/tree/master/keyboards/preonic)

## Burn Firmware

```shell
$ dfu-tool list
$ dfu-util -v -d 0483:df11 -a 0 -s 0x08000000:leave -D preonic_rev3_grumpycat.bin
```

## Layout

> Layer 0: classic alpha

![layer0](./preonic-layer0.png)

> Layer 1: punctuation

![layer1](./preonic-layer1.png)

> Layer 2: functions and numbers

![layer2](./preonic-layer2.png)

> Layer 3: Misc

![layer3](./preonic-layer3.png)
