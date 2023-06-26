# Packet injection for RTL8188-based WiFi devices on Android
This guide will show you how to get monitor mode and packet injection working on Android phones with WiFi adapters based on Realtek's RTL8188 chipset.

# WARNING
I'm not responsible for any damage done by using this guide. You're doing this on your own risk.

# Prerequesites
- RTL8188-based USB WiFi card. Most popular ones are TP-Link TL-WN722N (v2/v3) and TP-Link TL-WN725N
- Rooted Android phone with an unlocked bootloader
- Computer (or a VM) running Linux (most modern distros should work, but something based on Ubuntu 18.04 or newer is recommended)
- Source code of the kernel of your device, either official or unofficial
- Toolchain for compiling the kernel
- Patience
- (Optional) A little big of programming skills

# Dependencies
You're gonna need a few of dependencies, since recompiling the kernel is required. For Ubuntu-based distros this will be:

- `build-essential`
- `git`
- `linux-firmware`
- `nano` (or another favourite text editor)

# Cloning the kernel source
You will now have to download the source code of your device's kernel. You can either download and extract the ZIP or tarball, or clone the source from git.

# Entering the source code directory and setting up the toolchain and the compiling environment
TODO

# Adding the modified driver to kernel's source code
After making your device's defconfig, but before compiling the kernel, you will have to download and add the driver's code to the kernel. Follow these steps:

- Go to this directory using this command `cd drivers/net/wireless/realtek/`
- Download the driver `git clone --depth=1 https://github.com/Darkar25/realtek_rtwifi/ rtwifi/`
- Open the file called `Makefile` in your current directory, and add the following line after the other lines: `obj-$(CONFIG_RTWIFI) += rtwifi/`
- Open the file called `Kconfig` in your current directory, and add the following line after the other lines that start with "source": `source "drivers/net/wireless/realtek/rtwifi/Kconfig"`
- You're now done with adding the driver to the kernel's code, now you can return to the kernel source's root directory.

# Marking the driver for installation
For the following steps, you are going to be in the configuration menu called "menuconfig". Just follow these steps. The guide how to control this menu will be shown. To enter this menu, use this command: `make O=out ARCH=arm64 menuconfig`

## Enabling the mac80211 driver
The modified driver depends on the mac80211 feature of the kernel. To enable it, follow these steps.
```
[*] Networking support --->
        -*- Wireless --->
            <*> Generic IEEE 802.11 Networking Stack (mac80211)
```
Enable the feature, by highlighting it, and pressing space until a star symbol appears between the arrows, as shown above. Return to the main menu by double-pressing ESC until you appear at the main menu.

## Enabling the RTWIFI driver
Now you will enable the driver that you added into the kernel's source code before. Navigate to this location.
```
Device Drivers --->
    [*] Network device support --->
        [*] Wireless LAN --->
            [*] Realtek devices
                <*> RTL8723AU/RTL8188[CR]U/RTL819[12]CU (mac80211) support
```
Enable the feature the same way as before. Please note, that in most cases there will be doubled entries for this option! You will have to make sure that it is the correct one (RTWIFI) and not the unmodified one (RTL8XXXU)! Choosing both will cause serious trouble, and choosing the wrong one will not allow you to enable monitor mode and use packet injection. The correct one will probably be the second one, but to make sure, highlight one of them and press the question mark on your keyboard. If it says "CONFIG_RTWIFI" on the top it is the good one. If it says "CONFIG_RTL8XXXU" on the top, it is the wrong one! Once the correct one is checked, you can go to the main menu.

## Adding the network card's firmware into the kernel
We will now add the network card's firmware into the kernel, since Android doesn't have a way to add it without modifying the kernel. Navigate to this location.
```
Device Drivers --->
    Generic Driver Options --->
        -*- Userspace firmware loading support
        [*]     Include in-kernel firmware blobs in kernel binary
        (rtlwifi/rtl8188eufw.bin) External firmware blobs to build into the kernel binary
        (/lib/firmware) Firmware blobs root directory
```
You have to check the "Include in-kernel firmware blobs in kernel binary" option to unlock more features. Now, highlight the "External firmware blobs to build into the kernel binary" option and click Enter. Here, you will have to type the following `rtlwifi/rtl8188eufw.bin` and click Enter. "Firmware blobs root directory" should be left at `/lib/firmware` if you have the `linux-firmware` package installed, unless you have extracted the firmware in another place.

## Saving
Now that you have made all the changes required to the kernel, you can now save and exit from the menuconfig.

# Compiling the kernel
TODO

# Flashing the kernel
Assuming that the compilation succeeded, the compiled kernel will be in `out/arch/arm64/boot/` named `Image.gz`. To make a flashable ZIP, I extracted a AnyKernel3 installer for another custom kernel made for my device, replaced its Image.gz with the one compiled by me and zipped it back.

Now you will have to flash the kernel, look up the guides online for this, because the process may be different from device to device. Just remember to make backups of your original kernel!

# Testing
If you booted sucessfully with your modified kernel, you can now test if the monitor mode and packet injection work. Assuming you have Termux with sudo installed, you will now install the aircrack-ng suite using this guide https://github.com/pitube08642/aircrack-ng-for-termux. Connect the network card to your phone, and it should appear in `sudo iwconfig`, most likely as `wlan1`. To enable monitor mode, type in `sudo airmon-ng start wlan1`. To test injection, use `sudo aireplay-ng -9 wlan1`. To start monitoring nearby devices, use `sudo airodump-ng wlan1`.

# Issues I've encountered
aireplay-ng couldn't detect the channel of the WiFi card and thought it was -1. I had to manually set the channel of the card  using `sudo airodump-ng -c channel_number wlan1` and use aireplay-ng with `--ignore-negative-one` argument.

# Reporting issues
If you encounter any issues, submit an issue to this repo.