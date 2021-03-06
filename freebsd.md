# Battery

## Check status

```sh
apm
```

or

```sh
sysctl hw.acpi.battery
```

or

```sh
acpiconf -i 0
```

## Development Troubleshooting

_fatal error: 'X11/Xlib.h' file not_

The X11 library and header files can be found in `/usr/local/include` and `/usr/local/libs`.

# Boot

## Cannot boot to multiuser due to a segmentation fault in `fsck`

It happens when you poweroff your machine during booting. As a result
you'll get a segmentation fault during boot when `fsck` checks your hard drive.
As a result you'll be logged into a readonly single user mode.

1. Reboot.

2. Choose to boot in a single user mode.

3. Run `fsck`.

4. When prompted

   > **USE JOURNAL?**

   enter

   > *yes*

   It will not work probably. Try again and this time enter

   > *no*

5. Reboot.

## Dual boot

BunsenLabs GNU/Linux and FreeBSD using GRUB 2

1. Boot GNU/Linux.

2. Create `/boot/grub2/custom.cfg` file.

3. Add these lines to the file:

   ```text
   menuentry "FreeBSD" {
   set root='(hd0,3)'
   kfreebsd /boot/loader
   }
   ```

4. Run `grub2-mkconfig -o /boot/grub2/grub.cfg`.

### References

 * [GRUB2 Notes (scrobb.net)][boot-grub2]

[boot-grub2]: http://srobb.net/grub2.html

## Fix `run_interrupt_driven_hooks still waiting after 60 seconds for xpt_config`

Append those lines to the `/boot/device.hints` file:

```text
hint.ata.0.disabled="1"
hint.ata.1.disabled="1"
```

## Live CD boot

My old laptop decided to boot after setting **ACPI Support** and **Safe Mode**
options ON.

## Login in single user mode and make the file system writable

I get *Read-only file system* error when I boot into the single user mode.
According to [this website][boot-single-user-mode] you can simply run:

```sh
# Change the filesystem from read-only to write/read.
mount -u /
# Mount any other filesystems from /etc/fstab
mount -a
```

If the commands above fail then try to run `fsck -y` and try again.

[boot-single-user-mode]: https://lists.freebsd.org/pipermail/freebsd-questions/2004-November/066562.html

## So you cannot boot because you broke loader.conf, huh?

Let's say you've added `kern.vty=vt` to your `/boot/loader.conf` and now
your FreeBSD won't boot.

1. **Escape to loader prompt** in the boot menu.
2. Type `set kern.vty=""`.
3. Type `boot`.
4. Log in and remove that awful line from `loader.conf`.

# Console

## Turn off beep

FreeBSD > 11:

```sh
sysctl kern.vt.enable_bell=0
```

Older:

```sh
sysctl hw.syscons.bell=0
```

# Development

## Tracking the CURRENT / HEAD branch

```sh
rm -rf /usr/src
# Why this URL? See https://www.freebsd.org/doc/handbook/svn.html, A.3.6.
svn co https://svn.freebsd.org/base/head /usr/src
# ...
```

### References

- [Tracking -STABLE and -CURRENT (FreeBSD) (www.bsdnow.tv)][devel-bsdnow]
- [Scripts I use to live on the CURRENT branch][devel_dotfiles_freebsd_current]

[devel-bsdnow]: https://www.bsdnow.tv/tutorials/stable-current
[devel_dotfiles_freebsd_current]: https://github.com/0mp/dotfiles/blob/master/root_freebsd_current/setup

# Drives

## List all drives

There are a few different commands:

- `geom disk list`
- `gpart list`
- `usbconfig list`

## USB

### Eject (unmount) a USB drive

It is sufficient to simply unmount a USB drive with `umount`.

Reference:
http://unix.stackexchange.com/questions/35508/eject-usb-drives-eject-command

# Mouse

## Basic support

- Add to `/etc/rc.conf`:

  ```sh
  moused_enable="YES
  ```

- Add to `/usr/local/etc/X11/xorg.conf.d/11-synaptics-trackpad.conf`:

  ```
  Section "InputDevice"
        Identifier  "Touchpad0"
        Driver      "synaptics"
        Option      "Protocol" "psm"
        Option      "Device" "/dev/psm0"
        Option      "VertEdgeScroll" "off"
        Option      "VertTwoFingerScroll" "on"
        #Option      "HorizEdgeScroll" "off"
        #Option      "HorizTwoFingerScroll" "on"
        Option      "VertScrollDelta" "-111"
        #Option      "HorizScrollDelta" "-111"
        Option      "ClickPad" "on"
        #Option      "SoftButtonAreas" "4201 0 0 1950 2710 4200 0 1950"
        #Option      "AreaTopEdge" "5%"
  EndSection
  ```

  I am not sure if it is needed however.

- Add to `/boot/loader.conf`:

  ```sh
  hw.psm.synaptics_support="1"

  ```

## Natural scrolling

Set in `~/.Xmodmap`:

```
pointer = 1 2 3 5 4 6 7 8 9 10
```

### See also

- https://www.tomek.cedro.info/page/2/
- http://kartowicz.com/dryobates/2014-02/synaptics-freebsd/
- https://lists.freebsd.org/pipermail/freebsd-mobile/2016-April/013404.html

# Keyboard

## Set the key repeat rate

Add `xset r rate MS_DELAY RATE` to `.xinitrc`.

## Remap Caps Lock to Control

### In the console mode (tty)

Keymaps are stored in `/usr/share/vt/keymaps`.
**scan code** number 058 is represents the Caps Lock key.

Although it should be possible
[to load your custom keymap in `/etc/rc.conf`][kbd-custom-keymap] I failed to
do it. Instead I put
[`/usr/sbin/kbdcontrol -l ~/my-keyboard-mappings`][kbd-custom-keymap-bash] into
my `~/.bashrc`.

[kbd-custom-keymap]: https://www.geeklan.co.uk/?p=1274
[kbd-custom-keymap-bash]: http://www.freebsddiary.org/kbdcontrol.php

### In the X server

Add `setxkbmap -option ctrl:nocaps` to your `~/.xinitrc`.

## Set Polish keyboard in console

1. Add `keymap="pl.kbd"` to `/etc/rc.conf`.

## Set Polish keyboard in X

1. Add to the default class inside `/etc/login.conf`:

 ```
 :charset=UTF-8:\
 :lang=en_GB.UTF-8:
 ```

2. Run `cap_mkdb /etc/login.conf`.

# Network

## Temporary network device

```sh
ifconfig wlan0 create wlandev rum0
```

## Set up WiFi USB dongle

### TP-LINK TL-WN321G

Add `if_rum_load="YES"` to `/boot/loader.conf`.

### TP-LINK TL-WN725N v2.1

Add to `/boot/loader.conf`:

```text
if_rtwn_load="YES"
```

Add to `/etc/rc.conf`:

```text
wlans_rtwn0="wlan0"
ifconfig_wlan0="DHCP WPA"
```

You might need to restart `netif`.

Doesn't seem to work on eMac (powerpc).

## `wpa_supplicant.conf`

```text
ctrl_interface=/var/run/wpa_supplicant
network={
ssid="name"
psk="pass"
}
```

# Software installation

([Handbook reference][soft-handbook])

[soft-handbook]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-finding-applications.html


## Packages

* Check for known vulnerabilities.

  ```sh
  pkg audit -F
  ```

* Serach for packages.

  ```sh
  pkg search git
  ```

* List packages by origin.

  ```sh
  pkg search -o git
  ```

* Information about installed packages.

  ```sh
  pkg info pkg
  ```


## Ports

* Fetch the newest makefiles.

  ```sh
  portsnap fetch update
  ```

* Then to update all the old ports.

  ```sh
  portmaster -a
  ```

  **Don't do it.**

  It might take very long because

  > ```sh
  > sudo portmaster -a
  > ```
  >
  > This will update all of the ports on the system to their newest version.
  > Any new configuration options will be presented to you at the beginning of
  > the process. If you have any packages installed with pkg with newer versions
  > available through the ports system, these will be updated and transitioned
  > over to ports as well.

  ([Source][soft-ocean-ports])

  Consider using the `-d` option to automatically delete old files. Still
  awfully slow. ([Source][soft-d])

### See also

 - https://www.digitalocean.com/community/tutorials/how-to-install-and-manage-ports-on-freebsd-10-1


[soft-ocean-ports]: https://www.digitalocean.com/community/tutorials/how-to-install-and-manage-ports-on-freebsd-10-1
[soft-d]: https://lists.freebsd.org/pipermail/freebsd-questions/2012-July/243052.html

## Update software

```sh
freebsd-update fetch && freebsd-update install
pkg upgrade
```

([Source][soft-update])

[soft-update]: https://forums.freebsd.org/threads/15799/#post-93783

### Problems with loggining in after `freebsd-update install`.

Once, I ran `freebsd-update fetch && freebsd-update install` and I got an error
message from `cap_mkdb` I guess, but I cannot remember now. The problem was that
I was unable to login and use `su`/`sudo`.

I had to reboot with the power button and boot in single user mode. Then I had
to run `mount -u /; mount -a` to make the system writable (when you start up in
single user mode the filesystem is read-only).

At first I tried to run:

```sh
mergemaster
```

but it didn't solved the issue.

The I ran `freebsd-update rollback`. It didn't fix the issue with logging.

The problem was that `cap_mkdb` was unable to process the `/etc/login.conf` file
after the upgrade. Maybe the upgrade reset/broke `/etc/login.conf`. The error
was:

> ```text
> cap_mkdb: potential reference loop detected
> ```

when I tried to run `cap_mkdb /etc/login.conf`.

The solution was to edit `/etc/login.conf` so that there is no `:tc=default:` in
the definition of the `default` class (which obviosuly caused the referece
loop). Here's an example of a working `default` class from `/etc/login.conf` (as
you can see there is no `:tc=default:` there).

```text
default:\
        :passwd_format=sha512:\
        :copyright=/etc/COPYRIGHT:\
        :welcome=/etc/motd:\
        :setenv=MAIL=/var/mail/$,BLOCKSIZE=K:\
        :path=/sbin /bin /usr/sbin /usr/bin /usr/games /usr/local/sbin /usr/local/bin ~/bin:\
        :nologin=/var/run/nologin:\
        :cputime=unlimited:\
        :datasize=unlimited:\
        :stacksize=unlimited:\
        :memorylocked=64K:\
        :memoryuse=unlimited:\
        :filesize=unlimited:\
        :coredumpsize=unlimited:\
        :openfiles=unlimited:\
        :maxproc=unlimited:\
        :sbsize=unlimited:\
        :vmemoryuse=unlimited:\
        :swapuse=unlimited:\
        :pseudoterminals=unlimited:\
        :priority=0:\
        :ignoretime@:\
        :umask=022:
```

After fixing `/etc/login.conf` simply run `cap_mkdb /etc/login.conf`. Hopefully,
it does not produce any warnings. Now you should be able to log in again to your
FreeBSD.

# Sound

## Change volume

```sh
mixer vol +10
```

# System Administration Troubleshooting


## Broken `man -k`

`man -k`, `apropos` and `whatis` don't work. It looks like `whatis` database is
missing.

The solution is to run `makewhatis(1)`.

See also:

- `freebsd-update` and `whatis`: https://forums.freebsd.org/threads/53194/
- `/etc/periodic/`

## FUSE

## How to mount ext4?

```sh
pkg install fusefs-ext4fuse
kldload fuse.ko
mount /dev/da0p1 /mnt
```

See also:

- http://blog.ataboydesign.com/2014/04/23/freebsd-10-mounting-usb-drive-with-ext4-filesystem/
- https://www.freebsd.org/doc/handbook/usb-disks.html

## How to mount NTFS?

```sh
pkg install sysutils/fusefs-ntfs
kldload fuse
sysrc fusefs_enable=”YES” # Might not be needed.
ntfs-3g /dev/da0s1 /mnt/
```

See also:

- http://virtuallyhyper.com/2012/10/mounting-an-ntfs-disk-in-write-mode-in-freebsd-9/
- https://forums.freebsd.org/threads/mount-ntfs-windows-partition.5989/

# Printing

## Remote printer

### Configure `/etc/printcap`

Requirements:

 * `LPADDR=192.168.200.200` or `LPADDR=your-local-printer`

```
lp:\
        :rm=${LPADDR}:rp=raw:\
        :sh:\
        :mx#0:\
        :sd=/var/spool/lpd/lp:\
        :lf=/var/log/lpd-errs:
```

The `lpd` daemon has to be restarted.

### Print PDF files from a command line

```sh
pdf=file.pdf
ps=file.ps
pcl=file.pcl
printeraddr=192.168.0.13 # The name of the printer on the network.

# Printers prefer the PCL format.
pdf2ps "$pdf" "$ps"
cat "$ps" | gs -dSAFER -dNOPAUSE -q -sDEVICE=laserjet -sOutputFile=- - >"$pcl"

# If the lp service is configured then:
lpr "$pcl"
# otherwise it is possible to use netcat:
nc "$printeraddr" 9100 < "$pcl"
```

### References

 * https://www.freebsd.org/doc/handbook/printing.html
 * http://www.wonkity.com/~wblock/docs/html/lpdprinting.html

# Time and date

## Synchronize with the global time using ntp

Add `ntpd_sync_on_start="YES"` to  `/etc/rc.conf` so that ntpd sync with
the server on start.

([Source][time-1])

[time-1]: https://forums.freebsd.org/threads/16295/

## The time is wrong...

...although I've already tried setting `/etc/localtime` and running `tzsetup(8)`.

Well, then you should try to remove the `/etc/wall_cmos_clock` file.

([Source][time-2])

If you've already tried tinkering with `adjkerntz(9)`, `/etc/wall_cmos_clock` and
`tzsetup` then it's time to manually set up time with `date(1)`.

[time-2]: https://forums.freebsd.org/threads/48401/#post-271086

# User accounts

## Add a user

```sh
adduser
```

## Add a user to the wheel

```sh
sudo pw group mod wheel -m winniethepooh
```

## Install `sudo`

```sh
pkg install sudo
# Now, use `visudo` to uncomment the `%wheel ALL=(ALL) NOPASSWD: ALL` line to
# allow the wheel group to use `sudo(8)`.
visudo
```

## Modify user's full name

```sh
pw usermod winniethepooh -c "Winnie-the-Pooh"
```

# Video

## Brightness

There are a few ways to do it. `xrandr` will work in most cases though.

See also:

 * [Brightness (wiki.freebsd.org)](https://wiki.freebsd.org/Graphics/Brightness)

### Control brightness using `acpi_video` kernel module

1. Load the kernel module:

   ```sh
   kldload acpi_video
   ```

1. Use your hot keys to modify the brightness. `xbacklight(1)` might work as
   well.

### Modify `xorg.conf` to enable birghtness control

This one might work for Nvidia. Add the following lines to the `Device` section
of your `xorg.conf` file:

```text
Option      "RegistryDwords" "EnableBrightnessControl=1"
```

### Control brightness with `xrandr(1)`

```sh
xrandr --output HDMI-0 --brightness 0.5
```

### Control brightness using `xbrightness(1)`

1. Install:

   ```sh
   pkg install xbrightness
   ```

1. Use:

   ```sh
   xbrightness 65000
   ```

   Remember to use values close to 60k or you might end up with a black screen.

## Change the video mode

If the screen resolution is too small then you can change it using
these commands.

([Source][video-1])

```sh
# Load the vesa module.
kldload vesa
# Show available modes.
vidcontrol -i mode
# Choose one of the modes. 278 for example.
vidcontrol MODE_278
```

[video-1]: https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/consoles.html

# Window manager

## dwm

<strike>Awful.</strike> **Awesome.**

### Configuration files

- [agreif.blogspot.com][wm-dwm-1]
- [forums.freebsd.org][wm-dwm-2]
- [scrobb.net][wm-dwm-3]

[wm-dwm-1]: http://agreif.blogspot.com/2014/03/configure-dwm-on-freebsd.html
[wm-dwm-2]: https://forums.freebsd.org/threads/7816://forums.freebsd.org/threads/7816/
[wm-dwm-3]: http://srobb.net/dwm.html

## i3wm

### Installation

See [installation tuturial][wm-i3wm-install].

[wm-i3wm-install]: http://bottlenix.wikidot.com/installing-i3wm

### Configuration files

1. [giacomos][wm-i3wm-cf1]
2. [tunnelshade][wm-i3wm-cf2]
3. [i3wm.org][wm-i3wm-cf3]

[wm-i3wm-cf1]: https://github.com/giacomos/i3wm-config
[wm-i3wm-cf2]: http://blog.tunnelshade.in/2014/05/making-i3-beautiful.html
[wm-i3wm-cf3]: http://i3wm.org/docs/userguide.html#_scratchpad

## Mate

[Tutorial][wm-mate-1].

[wm-mate-1]: https://www.youtube.com/watch?v=YncqBz0bZcQ

## OpenBox

- [daemon-notes][wm-openbox-1]
- [forums.freebsd.org][wm-openbox-2]

[wm-openbox-1]: http://daemon-notes.com/articles/desktop/openbox
[wm-openbox-2]: https://forums.freebsd.org/threads/35308/

### tint2

If you want to run `tint2` on the startup the add a `tint2 &` line to your
autostart.sh.

([Source][wm-openbox-tint])

[wm-openbox-tint]: https://www.youtube.com/watch?v=k_-W8XmOycs

# Network

## Connect to eduroam

1. Add the following content to `/etc/wpa_supplicant.conf`:

   ```text
   ctrl_interface=/var/run/wpa_supplicant
   ctrl_interface_group=wheel
   network={
           ssid="eduroam"
           scan_ssid=1
           key_mgmt=WPA-EAP
           eap=TTLS
           identity="user@example.org"
           password="foobar"
           phase2="auth=MSCHAPV2"
   }
   ```

2. Append to `/etc/rc.conf`:

   ```text
   ifconfig_wlan0="WPA DHCP" # Change wlan0 to your interface.
   ```

3. Restart:

    ```sh
    service netif restart
    ```

([Source][net-eduroam])

[net-eduroam]: http://www.bishnet.net/tim/blog/2008/11/07/eduroam-on-freebsd/

### Resources

- [FreeBSD wireless quickstart][net-quickstart]
- FreeBSD Wireless Quickstart: https://srobb.net/fbsdquickwireless.html
- https://forums.freebsd.org/threads/43651/#post-243274
- http://lk.mimuw.edu.pl/index.php/linux
- https://www.freebsd.org/cgi/man.cgi?wpa_supplicant.conf

[net-quickstart]: https://srobb.net/fbsdquickwireless.html

# Video

## Allow brightness control.

Add `acpi_video_load="YES"` to `/boot/loader.conf`.

- ([Source 1][vid-1])
- ([Source 2][vid-2])

[vid-1]: https://forums.freebsd.org/threads/6186/
[vid-2]: https://forums.freebsd.org/threads/39771/

# The FreeBSD Project

## Using your @FreeBSD email alias with Gmail

Moved to https://wiki.freebsd.org/MateuszPiotrowski/UsingFreebsdEmailAliasWithGmail
