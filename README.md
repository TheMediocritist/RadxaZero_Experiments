# Radxa Zero firmware installation/update

## How to install Android on eMMC

1] Install AML Flash Tool 

Download from GitHub
```
git clone https://github.com/radxa/aml-flash-tool.git
```
Install the flash tool and dependencies
```
cd aml-flash-tool
./INSTALL.sh
```
2] Download Android image.
- Official Radxa image: https://dl.radxa.com/zero/images/android/
- monkaBlyat's image: https://drive.google.com/file/d/1FJ3kSpVpa-3YhfoykdoDTFwMDDnHPnKH/view?usp=sharing

3] Power up Radxa Zero in USB Boot mode.

Disconnect any hub and HDMI. Disconnect Zero. Press and hold (for a few seconds) USB boot button while reconnecting.

Check that the Zero is in usb boot/maskrom mode:

```
lsusb
```

You should see:
```
Bus 001 Device 052: ID 1b8e:c003 Amlogic, Inc. USB DEVICE
```

4] Flash the image to Zero eMMC:

```
aml-flash-tool.sh path_to_amlupdate_img/img_file
```

eg. 
```
aml-flash-tool.sh ../Downloads/android_9_radxa_zero_twrp_magisk.img
```
If successful you'll see:
```
Unpacking image [OK]
Initializing ddr ........[OK]
Running u-boot ........[OK]
Create partitions [OK]
Writing device tree [OK]
Writing bootloader [OK]
Wiping  data partition [OK]
Wiping  cache partition [OK]
Writing boot partition [OK]
Writing dtbo partition [OK]
Writing logo partition [OK]
Writing odm partition [OK]
Writing product partition [OK]
Writing recovery partition [OK]
Writing system partition [OK]
Writing vbmeta partition [OK]
Writing vendor partition [OK]
Resetting board [OK]
```

You can now reboot the Zero and it will boot to Android.
