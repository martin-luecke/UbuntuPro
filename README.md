# UbuntuPro

<<<<<<< HEAD
working:
* Booting (Kernel 4.10.0-35-generic)
* Keyboard, touchpad, and basic touchbar functionality (sometimes)
* screen + gpu
* opencl
* Screen brightness control
* Keyboard backlight
* Wifi with external adapter
* USB
* external display with HDMI adapter

not working:
* audio

Tricks that worked, shamefully stolen from another git:


## Keyboard/Touchpad/Touchbar

For this we need the drivers from https://github.com/roadrunner2/macbook12-spi-driver.git (a clone of https://github.com/cb22/macbook12-spi-driver which includes a preliminary touchbar driver and keyboard fixes). The following commands set this up.

First some extra packages:
```
sudo dnf install git kernel-devel dkms
```

Next we need to prepare for the modules to be included in the ramdisk (so they are loaded early during boot):
```
cat <<EOF | sudo tee /etc/dracut.conf.d/keyboard.conf
# load all drivers needed for the keyboard+touchpad
add_drivers+="applespi intel_lpss_pci spi_pxa2xx_platform appletb"
EOF
```
On distros using ```mkinitramfs``` instead of ```dracut``` you'll want to do the following instead:
```
cat <<EOF | sudo tee -a /etc/initramfs-tools/modules
# drivers for keyboard+touchpad
applespi
appletb
intel_lpss_pci
spi_pxa2xx_platform
EOF
```

Now get and build the drivers:
```
git clone https://github.com/roadrunner2/macbook12-spi-driver.git
pushd macbook12-spi-driver
git checkout touchbar-driver-hid-driver
sudo ln -s `pwd` /usr/src/applespi-0.1
sudo dkms install applespi/0.1
popd
```

Next we need to set the proper dpi for the touchpad (download the [61-evdev-local.hwdb](#file-61-evdev-local-hwdb) from this gist):
```
sudo cp ...the-downloaded-61-evdev-local.hwdb... /etc/udev/hwdb.d/61-evdev-local.hwdb
```

You can test the drivers by loading them and their dependencies:
```
sudo modprobe intel_lpss_pci spi_pxa2xx_platform applespi appletb
```

Finally, reboot to make sure it all works correctly:
```
sudo reboot
```

### Screen Brightness Control

Screen brightness control works out of the box on MacBookPro13,1 and MacBookPro13,2, but requires a kernel patch on MacBookPro13,3 (see also https://github.com/Dunedan/mbp-2016-linux/issues/2). The following will create and install the patched `apple-gmux`:
```
mkdir apple-gmux
pushd apple-gmux

curl -o apple-gmux.patch 'https://bugzilla.kernel.org/attachment.cgi?id=192601'
curl -o apple-gmux.c 'https://git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/plain/drivers/platform/x86/apple-gmux.c?id=refs/tags/v4.9.11'

patch < apple-gmux.patch

echo -e '
obj-m += apple-gmux.o

all:
\tmake -C /lib/modules/`uname -r`/build M=`pwd` modules
' > Makefile
make

mod=$(ls /lib/modules/`uname -r`/kernel/drivers/platform/x86/apple-gmux.ko*)
sudo mv $mod{,.orig}
sudo cp apple-gmux.ko /lib/modules/`uname -r`/kernel/drivers/platform/x86/
sudo depmod

popd

sudo reboot
```
=======
Installation:
http://linuxnewbieguide.org/how-to-install-linux-on-a-macintosh-computer/

Keyboard and Trackpad:
https://gist.github.com/roadrunner2/1289542a748d9a104e7baec6a92f9cd7

Wifi Stick:
info:
http://www.linux-hardware-guide.de/2012-10-07-edimax-ew-7811un-wireless-usb-150-mbits-802-11n

working fix:
https://adamscheller.com/systems-administration/rtl8192cu-fix-wifi/


Slack:
https://levlaz.org/installing-slack-on-ubuntu/

GPU Driver:
http://support.amd.com/en-us/kb-articles/Pages/AMD-Radeon-GPU-PRO-Linux-Beta-Driver%E2%80%93Release-Notes.aspx
>>>>>>> 95c29c713cc3a74357a3ebe770162da84778971d
