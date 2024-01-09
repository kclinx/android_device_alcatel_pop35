# android_device_ans_ul40
Android device configuration for ANS UL40 msm8909 

# Flashing info:
Get bkerler's edl program to flash the phone [here](https://github.com/bkerler/edl).

Use provided firehose in `firehose/msm8909.mbn`.

## DANGER AHEAD: !!!! THIS IS IMPORTANT !!!!, Please make a full backup of the stock rom before proceeding!

To put the phone into edl mode, disconnect it from power and turn it off.  Then hold both Volup+Voldn and plug it into your PC.  Hold the buttons until it vibrates once, the screen should stay off, or turn white, and it will come up in 9008 mode.

Then use bkerler's edl tool to take a backup.  This tool will also be the main flashing method.  It works great on linux.  On windows YMMV.
```
edl --loader=firehose/msm8909.mbn rl stock_rom_back
```
This will read all of the partitions from flash one-by-one, and store them in a directory.  Please keep the backup safe.  Without this you will brick your phone if you mess something up (which WILL happen).

# How to build
```
breakfast lineage_ul40-eng
mka bootimage
mka systemimage
```
This compiles the resulting: `out/target/product/ul40/{boot.img,system.img}`

# Does not boot out-of-the-box yet.  This is a hacky wip, don't expect anything to be functional.
Boots to lineageos 14 with precompiled kernel extracted from stock rom.
Instructions to use stock kernel:
```
# extract both lineage14 bootimg and stock bootimg
mkdir lineage-boot
cd lineage-boot
abootimg -x ../out/target/product/ul40/boot.img
cd ..
mkdir stock-boot
cd stock-boot
abootimg -x /path/to/stock/boot.img

# make modified bootimage with stock kernel
cd ../lineage-boot
abootimg --create precomp.img -f bootimg.cfg -k ../stock-boot/zImage -r initrd.img

# flash modified boot using edl
edl --loader=/path/to/ul40.mbn w boot precomp.img
```

With the compiled kernel, the screen doesn't work, but you can get debug output through UART.

Maybe with different kernel sources it would be possible to get it to work better.  You may need to dump the dtbs from the stock kernel to get it to do anything at all.

Use PabloCastellano's [extract-dtb](https://github.com/PabloCastellano/extract-dtb) tool to get the appended dtb image from the stock rom, and decompiled the dtbs using `dtc`.  This yielded 49 dts files, which I added to my forked kernel tree.
```
# unpack stock bootimage
$ abootimg -x boot.bin

# dump appended dtbs
$ extract-dtb -o stock_dtbs zImage

# decompile each dtb to a dts
$ mkdir dump_dts
$ for i in `ls stock_dtbs/*.dtb` ; do dtc -I dtb -O dts -o dump_dts/$(basename -s .dtb $i).dts $i ; done
```
Then add the .dts files to arch/arm/boot/dts/qcom, and edited the Makefile to build them for `CONFIG_ARCH_MSM8909`.

ans/teleepoch enabled `/proc/config.gz` in the stock kernel, so you can also dump that using adb and get a defconfig for the device.

By probing the motherboard with a multimeter, I identified the pinout of the UART port too.  It is a 1.8V UART and runs at `115200` baud. Access to the uart port is disabled in the stock kernel, as the port is shared with the bluetooth chip, however you can use it for debugging a custom kernel or seeing bootloader logs. ![uart](images/uart.jpg)

## What works (with prebuilt kernel):
 * Wifi (after wlan.ko from stock rom is loaded)
 * Display
 * Touchscreen
 * Vibration

## What doesn't work
 * everything else

## Lineage running with stock kernel:
![lineage wip](images/lineage.jpg)
