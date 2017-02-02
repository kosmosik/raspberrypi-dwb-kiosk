# raspberrypi-dwb-kiosk
Simple restricted web kiosk based on Raspberry Pi SBC, Raspbian and dwb web browser.

## Required skills

This guide assumes you know your way around Linux systems, Rasberry Pi SBC and you can use text console and tools on Debian based Linux system.

Detailed step by step instructions are NOT in scope of this guide.

## System preparation

1. Download and install latest Raspbian image to SD card. I recommend starting with "lite" image.
2. Boot the system and let it resize filesystem to SD card size (it does that automagically).
3. Consider converting the SD card to `F2FS` filesystem as described in [here](https://movr0.com/2016/08/19/convert-raspberry-pi-123-to-f2fs/).
4. Consider using `tmpfs` (a ramdisk basically) for `/tmp`, `/var/log`, `/var/spool` and `/var/tmp` directories to minimise SD card writes. Some hints [here](https://www.domoticz.com/wiki/Setting_up_a_RAM_drive_on_Raspberry_Pi).
5. Consider uninstalling all not required packages from system. Nice tool that eases this task is `debfoster`.
6. Install and configure required network services:
..* configure network interfaces (wired or wireless) to your liking,
..* install and enable OpenSSH daemon (`apt-get install ssh`) to allow remote shell access
..* install and enable NTP daemon (`apt-get install ntp ntpdate`)
7. Install all required software (`apt-get install xorg dwb ratpoison xautolock`)
