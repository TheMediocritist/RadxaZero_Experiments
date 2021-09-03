# Radxa Zero firmware installation/update

This is mainly for my reference, to put instructions all in one place until the wiki is fleshed out.
Some useful links (that I've grabbed bits from):

- Radxa Dev guide: https://github.com/radxa/documentation/tree/master/rs102
- https://forum.radxa.com/t/introduce-the-radxa-zero/6550/145
- eMMC AML Flash Tool instructions: https://wiki.radxa.com/Zero/install/eMMC_aml_tool
- Official firmware & boot loaders: https://dl.radxa.com/zero/

Other Linux images:
- Manjaro: https://github.com/manjaro-arm/radxa-zero-images/releases/tag/20210830
- TwisterOS Beta 4: https://drive.google.com/file/d/1aqB7A_EuLGvL7pR8LhP6RatikxvJyy6i/view?usp=sharing
- 

### How to install Android on eMMC

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

# Image writing tools

- Option 1 - Balena Etcher: https://www.balena.io/etcher/
- Option 2 - USB Imager: https://gitlab.com/bztsrc/usbimager/tree/binaries
- Option 3 - dd (linux only)



## Bootloader to SD Card (Board with no eMMC)

Download the pre-prepared u-boot from dl.radxa:
```
wget https://dl.radxa.com/zero/images/loader/u-boot.bin.sd.bin
```
Copy Twister image to SD card:

```
dd if=Twister-OS_Armbian_Focal_20.10_xfce-rockpi-zero-beta2.img of=/dev/sdh bs=1M status=progress
```

After this is finished , copy the downloaded u-boot.bin.sd.bin to the SD card (in two steps):
```
dd if=u-boot.bin.sd.bin of=/dev/sdh conv=fsync,notrunc bs=1 count=444
dd if=u-boot.bin.sd.bin of=/dev/sdh conv=fsync,notrunc bs=512 skip=1 seek=1
```

## Make a bootable SD card image (eMMC)
I'm not sure if you can access the SD card in a Zero with eMMC using the USB Boot method.

Some images can be imaged to SD using Balena Etcher and will boot just fine. These include:
- Official Radxa Armbian
- TwisterOS

Others such as 
Experiment
