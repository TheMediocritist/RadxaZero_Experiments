# Radxa Zero firmware installation/update

This is mainly for my reference, to put instructions all in one place until the wiki is fleshed out.
Some useful links (that I've grabbed bits from):

- Radxa Dev guide: https://github.com/radxa/documentation/tree/master/rs102
- https://forum.radxa.com/t/introduce-the-radxa-zero/6550/145
- eMMC AML Flash Tool instructions: https://wiki.radxa.com/Zero/install/eMMC_aml_tool
- Official firmware & boot loaders: https://dl.radxa.com/zero/

Other Linux images:
- Manjaro: https://github.com/manjaro-arm/radxa-zero-images/releases/
- TwisterOS Beta 4: https://drive.google.com/file/d/1aqB7A_EuLGvL7pR8LhP6RatikxvJyy6i/view?usp=sharing


### How to completely erase Android from eMMC

Hold the USB Boot button and connect Zero to Linux PC.

lsusb should show the followling VID/PID:
```
Bus 002 Device 030: ID 1b8e:c003 Amlogic, Inc. GX-CHIP
```
This means the Zero is in USB Boot mode

Use pyamlboot to load the fastboot loader from USB

pyamlboot is a tool by @superna9999 for Amlogic USB Boot. To install it:

Install pyamlboot tool
```
pip3 install pyamlboot
```
pip3 install pyamlboot

Download the prebuilt fastboot loader:
```
wget https://dl.radxa.com/zero/images/loader/rz-fastboot-loader.bin
boot-g12.py rz-fastboot-loader.bin
```
It should output
```
Firmware Version :
ROM: 3.2 Stage: 0.0
Need Password: 0 Password OK: 1
Writing rz-fastboot-loader.bin at 0xfffa0000...
[DONE]
Running at 0xfffa0000...
[DONE]
AMLC dataSize=16384, offset=65536, seq=0...
[DONE]
AMLC dataSize=49152, offset=393216, seq=1...
[DONE]
AMLC dataSize=16384, offset=229376, seq=2...
[DONE]
AMLC dataSize=49152, offset=245760, seq=3...
[DONE]
AMLC dataSize=49152, offset=294912, seq=4...
[DONE]
AMLC dataSize=16384, offset=65536, seq=5...
[DONE]
AMLC dataSize=1406320, offset=81920, seq=6...
[DONE]
[BL2 END]
```
Now the Zero is in fastboot mode.
```
lsusb
Bus 002 Device 032: ID 18d1:fada Google Inc. USB download gadget  Serial: AMLG12A-RADXA-ZERO
```
Use fastboot to erase the eMMC
```
$ fastboot devices
AMLG12A-RADXA-ZERO	fastboot
$ fastboot erase bootloader
Erasing 'bootloader'                               OKAY [  3.601s]
Finished. Total time: 3.613s
$ fastboot erase bootenv
fastboot erase bootenv
Erasing 'bootenv'                                  OKAY [  0.097s]
Finished. Total time: 0.110s
```
Now erase the eMMC user partition from offset 0:
```
fastboot erase 0
Erasing '0'                                        OKAY [ 17.574s]
Finished. Total time: 17.586s
```
The eMMC is completely wiped now, unplug and plug the USB C cable. Insert a bootable uSD card, the Zero should boot from it.

### How to install boot loader on a wiped eMMC

Reboot, holding USB boot button.
```
sudo boot-g12.py rz-udisk-loader.bin
wget https://dl.radxa.com/zero/images/loader/u-boot.bin
sudo dd if=u-boot.bin of=/dev/sdx bs=512 seek=1
```

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

## Write images to eMMC from PC

Boot the Zero to USB Boot mode

Download and run the prebuilt USB Disk loader
```
wget https://dl.radxa.com/zero/images/loader/rz-udisk-loader.bin
boot-g12.py rz-udisk-loader.bin
lsusb
Bus 002 Device 004: ID 18d1:fada Google Inc. USB download gadget Serial: AMLG12A-RADXA-ZERO
```
The PC should show a USB Disk device, which is the eMMC of Zero.


## Write bootloader to eMMC
Write the u-boot.bin to eMMC
```
sudo dd if=u-boot.bin of=/dev/sdx of=/dev/sdx bs=512 seek=1
```
Unplug and plug USB C, Zero should boot from u-boot in eMMC



## Write bootloader to SD Card (Board with no eMMC)

Download the pre-prepared u-boot from dl.radxa:
```
wget https://dl.radxa.com/zero/images/loader/u-boot.bin.sd.bin
```
Find the id of your SD card.
```
lsblk
```
!! Make sure you have correctly identified your SD card, otherwise you might bork your PC's linux install!!

Copy disk image to SD card:
```
dd if=path_to_disk_img/img_file.img of=/dev/your_device_id bs=1M status=progress
```
eg.
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
Others such as...

### Experiments

A couple of my mini linux handheld projects use non-standard displays. Neither of these works without some effort:
- Waveshare 3.5 inch 480x800 display
- Waveshare 7.9 inch 400x1280 display

## Edit display EDID binary files

Option 1: Create EDID from modeline (linux)
```
cvt 480 800 60
```
Copy the modeline generated by CVT
```
sudo apt install zsh edid-decode automake dos2unix
git clone https://github.com/akatrevorjay/edid-generator
cd edid-generator
./modeline2edid - <<< 'Modeline "480x800" 40.540 480 524 612 640 800 810 816 1056 +hsync +vsync'
make
```

Error. edid-generator can't handle the odd display layout.
```
./modeline2edid - <<< 'Modeline "480x800" 40.540 480 524 612 640 800 810 816 1056 +hsync +vsync ratio=3:5'
```
Doesn't work.

Option 2: Extract and modify existing EDID from display
- Download AW EDID Editor (windows/mac only):
- https://www.analogway.com/americas/products/software-tools/aw-edid-editor/
- Plug display in and extract registry edid.
- Modify - EDID 1.4 allows setting the portrait/landscape flag <- this is promising!
- Save 480x800.edid

Option 3: extract existing EDID from display (linux)
Install read-edid and edid-decode tools
```
sudo apt install read-edid edid-decode

sudo find /sys/devices/pci*/*/*/*/*/*HDMI* -name "*edid*" | head -1 | xargs -I{} cp {} edid.bin
```
Edit/patch with wxEDID utility
```
$ sudo apt install -y libwxgtk3.0-dev
$ wget https://sourceforge.net/projects/wxedid/files/wxedid-0.0.19.tar.gz/download
$ tar xvf wxedid-0.0.19.tar.gz
$ cd wxedid-0.0.19
$ ./configure --prefix=$HOME
$ make
$ make install
```

Add the edid file to linux firmware folder
```
sudo mkdir /usr/lib/firmware/edid
sudo cp 480x800.bin /usr/lib/firmware/edid
```

Now load it at boot
Edit the config file:
```
sudo nano /boot/extlinux/extlinux.conf
```
Add this to the last line (after APPEND [.....])
```
drm_kms_helper.edid_firmware=edid/480x800.bin
reboot
```

## For X based linux
Find out wtf went wrong
```
sudo nano /var/log/Xorg.0.log
dmesg -H
journalctl -b -g "edid" --no-hostname --no-pager
```

Something like this...
```
Section "Monitor"
   Identifier "HDMI-1"
   #VendorName "Monitor Vendor"
   #ModelName "Monitor Model"
   Modeline "480x800b" 40.5 480 508 598 640 800 810 816 1056 -hsync +vsync
   HorizSync 24.0 - 94.0
   VertRefresh 24.0 - 87.0
EndSection

Section "Device"
   Identifier "Device0"
   #Driver "modesetting"
   Option "UseEDID" "false"
EndSection

Section "Screen"
   Identifier "Screen0"
   Device "Device0"
   Monitor "HDMI-1"
   DefaultDepth 24
   Option "DPI" "270 x 270"
   #Option "ModeValidation" "AllowNonEdidModes"
   Option "ModeValidation" "AllowNonEdidModes, NoMaxPClkCheck, NoEdidMaxPClkCheck, NoMaxSizeCheck, NoHorizSyncCheck, NoVertRefreshCheck, NoVirtualSizeCheck, NoTotalSizeCheck, NoExtendedGpuCapabilitiesCheck" 
   Option "metamodes" "HDMI-1: 480x800_60.00 +0+0"
   SubSection "Display"
     Depth 24
     Modes "480x800b"
     Modes "480x800_60.00"
   EndSubSection
EndSection
```
Edit xorg configuration to specify monitor details
```
sudo nano /etc/X11/xorg.conf.d/10-Monitors.conf
```

Find edids:
```
sudo find /. |grep -i edid
```
Some interesting extras:
```
/./sys/kernel/debug/dri/0/HDMI-A-1/edid_override
/./sys/kernel/debug/dri/0/Composite-1/edid_override
/./sys/devices/platform/soc/ff900000.vpu/drm/card0/card0-HDMI-A-1/edid
/./sys/devices/platform/soc/ff900000.vpu/drm/card0/card0-Composite-1/edid
/./sys/module/drm_kms_helper/parameters/edid_firmware
/./sys/module/drm/parameters/edid_firmware
/./sys/module/drm/parameters/edid_fixup
```


EDID vs X11 format
```
Please note that the EDID data structure expects the timing
values in a different way as compared to the standard X11 format.

X11:
HTimings:  hdisp hsyncstart hsyncend htotal
VTimings:  vdisp vsyncstart vsyncend vtotal

EDID:
#define XPIX hdisp
#define XBLANK htotal-hdisp
#define XOFFSET hsyncstart-hdisp
#define XPULSE hsyncend-hsyncstart

#define YPIX vdisp
#define YBLANK vtotal-vdisp
#define YOFFSET (63+(vsyncstart-vdisp))
#define YPULSE (63+(vsyncend-vsyncstart))
```
Source:
```
/usr/share/doc/kernel-doc-3.11.4/Documentation/EDID
```

Convert EDID in text format to .bin format
```
xxd -r -p text_dump > binary_dump
```
Check that it worked:
```
cat 400x1280v14.bin | edid-decode
```

Tests
```
cat /sys/class/drm/card0-HDMI-A-1/status
cat /sys/class/drm/card0-HDMI-A-1/modes
cat /sys/class/drm/card0-HDMI-A-1/enabled
cat /sys/class/drm/card0-HDMI-A-1/edid
cat /sys/class/graphics/fb0/virtual_size
cat /sys/class/graphics/fb0/modes
cat /sys/class/drm/card0-HDMI-A-1/edid | edid-decode
drm_info
```

Detailed timings modes from edid are not available in DRM (per drm_info). 


Using modetest and proptest
Compile and install these two tools
```
git clone https://github.com/freedesktop/mesa-drm
cd mesa-drm
meson builddir/
ninja -C builddir/
sudo cp builddir/tests/modetest/modetest /usr/bin
sudo cp builddir/tests/proptest/proptest /usr/bin
```

Running xrandr through ssh:
```
DISPLAY=:0 xrandr
DISPLAY=:0 xrandr --output HDMI-1 --mode <width>x<height>
DISPLAY=:0 xrandr --newmode "480x800_60.00" 40.540 480 524 612 640 800 810 816 1056 +hsync +vsync
DISPLAY=:0 xrandr --addmode HDMI-1 "480x800_60.00"
DISPLAY=:0 xrandr -s 480x800_60.00
DISPLAY=:0 xrandr --output HDMI-1 --mode 480x800_60.00
```

Running swaymsg through ssh:
Run locally:
```
sway --get-socketpath
```
Then use the nominated socketpath to run swaymsg through ssh:
```
DISPLAY=:0 swaymsg -s /run/user/1000/sway-ipc.1000.1045.sock -t get_outputs -r
DISPLAY=:0 swaymsg -s /run/user/1000/sway-ipc.1000.1045.sock -- output HDMI-A-1 mode --custom 480x800@60Hz
