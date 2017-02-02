# raspberrypi-dwb-kiosk

Simple restricted web kiosk based on [Raspberry Pi SBC](https://www.raspberrypi.org/), [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) and [dwb](https://portix.bitbucket.io/dwb/) web browser.

## Required skills

This guide assumes you know your way around Linux systems, Rasberry Pi SBC and you can use text console and tools on Debian based Linux system. Detailed step by step instructions are NOT in scope of this guide.

## Files

Most important config files and scripts are in `filesystem` directory in this repo. Refer to them as they are mentioned in this guide. DO NOT COPY THE files to your system without inspecting them and understanding what they do since it will probably don't work for you that way.

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
11. Install additional fonts if needed.

### Automatic hostname

A script `/etc/network/if-pre-up.d/00-hwaddr-to-hostname` is provided to change hostname to contain last three octets of hardware address (e.g. `kiosk-14d28b`) to allow easy network identification of the device. Script is run prior to starting DHCP client on network interfaces so your DHCP server will register unique kiosk name.

### Boot config files

If you run into trouble getting video output check options in `/boot/config.txt`. Refer to [this article](http://elinux.org/RPiconfig).

### Silent boot

If you wish to have completely silent (e.g. no information, scrolling letters, logos, etc.) boot refer to `/boot/cmdline.txt` file options. You can also disable getty on first console.

### Directory structure and user creation

We will run kiosk user session as `kiosk` user.

Create directory structure:

`/opt/bin` - kiosk scripts go here

`/opt/etc` - kiosk config files go here

`/opt/home/kiosk` - this is kiosk user home directory

Create kiosk user with `/opt/home/kiosk` as home directory. Chown `/opt/home/kiosk` to kiosk user.

## Display manager

Kiosk uses [nodm](https://github.com/spanezz/nodm) display manager to start XOrg under given user. Check `/etc/defaults/nodm` for configuration. This service needs to be enabled and started on boot.

## Xinit

Display manager starts XOrg as user `kiosk` and starts `/opt/bin/xinit-kiosk` script. This script controls execution of kiosk session.

### xautolock

Xinit script launches [xautolock](https://linux.die.net/man/1/xautolock) program. `xautolock` implements session reseting after inactivity. After 1 minute of inactivity `xautolock` sends command to `dwb` to close all but one tabs and load start page.

## Window manager

Kiosk uses [Ratpoison](http://www.nongnu.org/ratpoison/) as fullscreen window manager. Ratpoison is used only to display single application fullscreen and also catch some keys pressed by the user. Ratpoison is configured in '/opt/etc/ratpoisonrc'. Refer to [ratpoison manual](http://www.nongnu.org/ratpoison/doc/)

### Window manager defined keybindings

Ratpoison config implements few keybindings:

`F1` key - send command to `dwb` to open home page

`F5` key - send command to `dwb` to reload current tab

`Control+F12` keys - send command to `dwb` to quit (thus ending X session, thus restarting it via nodm)

## Web browser

Kiosk uses [dbw](https://portix.bitbucket.io/dwb/) web browser to display content. `dwb` is started from `xinit` and controlled with `dwbremote`. Refer to [dwb manual](https://portix.bitbucket.io/dwb/resources/manpage.html).

### Dwb configuration

`dwb` configuration files inside `kiosk` user's home directory are linked to files in `/opt/etc/dwb` directory. The links need to be made as `root` (or in other way not writeable by `kiosk` user).

Two main files are used:

`/opt/etc/dwb/keys` - this file defines (unmaps) all keyboard shortcuts - all of them are removed from user

`/opt/etc/dwb/settings` - this file configures `dwb` behavior - tune it to your liking.

### Security considerations

In theory the user `kiosk` has no access to shell. When setting up the browser consider all possible scenarios when the user can break out of this. Pay attention to any helper programs like downloaders or scheme (e.g. mailto:, telnet: etc.) handlers. All such commands (even if unused) should be substituted with harmless `/bin/true` program. All access to local `file://` protocol should be disabled. All forms of local cache (e.g. HTML5 cache) should be disabled.

Kiosk browser is not restricted to specific URL or domain. You can lock the browser with `dlock` and `durl` commands. Refer to manual.

Tabbed browsing is enabled. User can open a new tab by right clicking or `Control`-clicking an link element. This behavior can't be turned off in `dwb`. It can be mitigated by using `xmodmap` to disable right mouse click and Control keys entirely.

If you wish to restrict the user to specific sites the best way to do it would be on the network level. Use local or remote HTTP proxy with implemented access control list.

## Visual appearance

Appearance of kiosk can be tuned to some extent.

* Changing GTK-3 theme (aplies only to scrollbars in `dwb`)
* Changing cursor theme
* Changing colors and fonts in `dwb` settings
