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

# Does not boot out-of-the-box yet.
Boots to lineageos 14 with precompiled kernel extracted from stock rom.
Instructions:
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

## What works:
 * Wifi (after wlan.ko from stock rom is loaded)
 * Display
 * Touchscreen
 * Vibration

## What doesn't work
 * everything else
