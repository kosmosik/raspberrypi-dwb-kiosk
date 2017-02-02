# raspberrypi-dwb-kiosk
Simple restricted web kiosk based on Raspberry Pi SBC, Raspbian and dwb web browser.

## Required skills

This guide assumes you know your way around Linux systems, Rasberry Pi SBC and you can use text console and tools on Debian based Linux system. Detailed step by step instructions are NOT in scope of this guide.

## Files

Most important config files and scripts are in `filesystem` directory in this repo. Reffer to them as they are mentioned in this guide. DO NOT COPY THE files to your system without inspecting them and understanding what they do since it will probably don't work for you that way.

## System preparation

1. Download and write latest Raspbian image to SD card. I recommend starting with "lite" image.
2. Boot the system and let it resize filesystem to SD card size (it does that automagically).
3. Update system (`apt-get update && apt-get upgrade`).
4. Consider converting the SD card to `F2FS` filesystem as described in [here](https://movr0.com/2016/08/19/convert-raspberry-pi-123-to-f2fs/).
5. Consider using `tmpfs` (a ramdisk basically) for `/tmp`, `/var/log`, `/var/spool` and `/var/tmp` directories to minimise SD card writes. Some hints [here](https://www.domoticz.com/wiki/Setting_up_a_RAM_drive_on_Raspberry_Pi).
6. Consider uninstalling all not required packages from system (`debfoster` is a handy tool that allows that).
7. Configure network interfaces (wired or wireless) to your liking.
8. Install, configure and enable OpenSSH daemon (`apt-get install ssh`) to allow remote shell access.
9. Install, configure and enable NTP daemon (`apt-get install ntp ntpdate`) to enable network time synchronization.
10. Install all required software (`apt-get install xorg dwb ratpoison xautolock`).

## Boot config files

If you run into trouble getting video output check options in `/boot/config.txt`. Reffer to [this article](http://elinux.org/RPiconfig).

### Silent boot

If you wish to have completely silent (e.g. no information, scrolling letters, logos, etc.) boot reffer to `/boot/cmdline.txt` file options. You can also disable getty on first console.

## Directory structure and user creation

We will run kiosk user session as `kiosk` user.

Create directory structure:

`/opt/bin` - kiosk scripts go here

`/opt/etc` - kiosk config files go here

`/opt/home/kiosk` - this is kiosk user home directory

Create kiosk user with `/opt/home/kiosk` as home directory. Chown `/opt/home/kiosk` to kiosk user.

## Display manager

Kiosk uses `nodm` display manager to start XOrg under given user. Check `/etc/defaults/nodm` for configuration. This service needs to be enabled and started on boot.

## Xinit

Display manager starts XOrg as user `kiosk` and starts `/opt/bin/xinit-kiosk` script. This script controls execution of kiosk session.

## Window manager

Kiosk uses `ratpoison` as fullscreen window manager. Ratpoison is used only to display single application fullscreen and also catch some keys pressed by the user. Ratpoison is configured in '/opt/etc/ratpoisonrc'.

### Window manager defined keybindings

Ratpoison config implements few keybindings:

`F1` key - send command to dwb to open home page
`F5` key - send command to dwb to reload current tab
`Control+F12` keys - send command to dwb to quit (thus ending X session, thus restarting it via nodm)






