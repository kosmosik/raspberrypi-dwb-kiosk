# raspberrypi-dwb-kiosk

Simple web kiosk based on [Raspberry Pi](https://www.raspberrypi.org/), [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) and [dwb](https://portix.bitbucket.io/dwb/) web browser.

## Required skills

This guide assumes you can use text console and admin tools on Debian based Linux system.

Detailed step by step instructions are NOT scope of this guide.

This guide gives just overall information on how to set it up.

The rest is up to you.

## Files

Most important config files and scripts are provided in `filesystem/` directory in this repo. DO NOT COPY these files to your system without inspecting them and understanding what they do.

## Preparation

1. Download and write latest Raspbian "lite" image to SD card.
2. Boot and let it resize filesystem to SD card size (it does that automagically).
3. Update system: `apt-get update && apt-get upgrade`.
4. Consider converting the SD card to `F2FS` filesystem as described in [here](https://movr0.com/2016/08/19/convert-raspberry-pi-123-to-f2fs/).
5. Consider using `tmpfs` (ramdisk basically) for `/tmp`, `/var/log`, `/var/spool` and `/var/tmp` directories to minimise SD card writes. Some hints [here](https://www.domoticz.com/wiki/Setting_up_a_RAM_drive_on_Raspberry_Pi).
6. Consider uninstalling all not required packages from system. `debfoster` is a handy tool that automates that.
7. Configure network interfaces to your liking.
8. Install and enable OpenSSH daemon to allow remote shell access: `apt-get install ssh`.
9. Install and enable NTP daemon to enable network time synchronization: `apt-get install ntp ntpdate`.
10. Install all required software: `apt-get install xorg nodm ratpoison dwb xautolock`.
11. Install additional fonts if needed.

### Hostname change script

A script `/etc/network/if-pre-up.d/00-hwaddr-to-hostname` is provided to change hostname to contain last three octets of hardware address (e.g. `kiosk-14d28b`) to allow easy network identification of the device. Script is run prior to starting DHCP client on network interfaces so your DHCP server will register unique kiosk name.

### Boot config files

If you run into trouble getting video output change options in `/boot/config.txt`. Refer to [this article](http://elinux.org/RPiconfig).

### Silent boot

If you wish silent boot (e.g. no information, scrolling letters, logos, etc.) refer to `/boot/cmdline.txt` file options.

You can also disable getty on first console and tinker with `printk` to shush noisy kernel modules.

### Directory structure

Create directory structure:

`/opt/bin` - scripts go here

`/opt/etc` - config files go here

`/opt/home/kiosk` - `kiosk` user's home directory

### User creation

Kiosk user session runs as `kiosk` user.

Create `kiosk` user with `/opt/home/kiosk` as home directory. Don't forget to chown `/opt/home/kiosk`!

## Display manager setup

Kiosk uses [nodm](https://github.com/spanezz/nodm) display manager to start X as `kiosk` user.

Refer to `/etc/defaults/nodm` for configuration.

This service needs to be enabled and started on boot.

Restart `nodm` after any changes to X session configuration.

## Xinit

Display manager starts X as user `kiosk` and executes `/opt/bin/xinit-kiosk` script.

This script controls execution of kiosk session.

### xautolock

`Xinit` script executes [xautolock](https://linux.die.net/man/1/xautolock) program.

After 1 minute of inactivity `xautolock` sends command to `dwb` to close all but current tab and load start page (`only;;home`).

## Window manager

Kiosk uses [Ratpoison](http://www.nongnu.org/ratpoison/) as window manager.

Ratpoison is used to display single application in fullscreen and catch keys pressed by the user.

Ratpoison is configured in `/opt/etc/ratpoisonrc`. Refer to [ratpoison manual](http://www.nongnu.org/ratpoison/doc/)

### Window manager defined keybindings

Ratpoison config implements few useful keybindings:

`F1` key - send command to `dwb` to open home page

`F5` key - send command to `dwb` to reload current tab

`Control+F12` keys - send command to `dwb` to quit (thus restarting X session via `nodm`)

## Web browser

Kiosk uses [dbw](https://portix.bitbucket.io/dwb/) web browser to display content.

`dwb` is started from `xinit` and controlled with `dwbremote`. Refer to [dwb manual](https://portix.bitbucket.io/dwb/resources/manpage.html).

### Dwb configuration

`dwb` configuration files inside `kiosk` user's home directory are linked to files in `/opt/etc/dwb` directory.

The links need to be owned by `root` or in other way made not writeable by `kiosk` user.

Two main files are used:

`/opt/etc/dwb/keys` - defines keyboard shortcuts - all are removed

`/opt/etc/dwb/settings` - configures `dwb` behavior - tune it to your liking

### Tabbed browsing

Tabbed browsing is enabled. User can open a new tab by right clicking or Control-clicking an link element.

This behavior can't be disabled in `dwb`. It can be mitigated by using `xmodmap` to disable right mouse click and Control keys entirely.

### Visual appearance

Appearance can be tuned to some extent:
* GTK-3 theme which aplies only to scrollbars in `dwb`
* colors and fonts in `dwb` settings
* X cursor theme

## Security considerations

In theory the `kiosk` user has no access to shell. In fact has no means to launch any terminal emulator or filesystem browser or any other program whatsoever.

When setting up `dwb` browser consider all possible scenarios in which the user could break out and start any unwanted program.

Pay attention to external helper programs or scheme handlers. All such commands even unused should be set to harmless `/bin/true`.

All access to `file://` scheme should be disabled.

All forms of persistent storage (e.g. HTML5 storage) should be disabled.

### Access control

Kiosk browser is not restricted to specific URL or domain.

You can lock the browser with `dlock` and `ulock` commands. Refer to manual.

If you wish to restrict user to specific sites the best way to do it would be to use local or remote HTTP proxy with access control lists.
