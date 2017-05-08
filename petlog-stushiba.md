---
tags: [desktop, x64, archlinux]
---

# From installation media

closely following https://wiki.archlinux.org/index.php/Beginners%27_guide

## as root outside chroot (essentially, from memory)

- `blkid`
- parted: add 512mb fat32 partition followed by 513MB-100% ext4 partition
- mkfs.fat for ESP, mkfs.ext for root (with label 'stushiba')
- label ESP 'ESP'
- dhcpcd messing around due to wifi stick not being in kernel
- `cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak`
- `pacman -S reflector`
- `reflector --verbose -c US -f 200 --sort score -p https --save /etc/pacman.d/mirrorlist`
- `mount /dev/sdc2 /mnt; mkdir -p /mnt/boot; mount /dev/sdc1 /mnt/boot`
- pacstrap a bunch of stuff:
  - base, base-devel
  - git
  - gnome
  - weston
  - chromium
  - firefox
  - mpv
  - deluge
  - refind-efi
- choose xf86-input-libinput when prompted since it looks like that's the future
- everything else is defaults

## in chroot (from `sudo cat /root/.bash_history`)

- uncomment `en_US.UTF-8` in /etc/locale.gen
- `echo 'LANG=en_US.UTF-8' >/etc/locale.conf`
- tzselect (thinking it would do something I didn't already know)
- `hwclock --systohc --utc`
- `ln -s /usr/share/zoneinfo/America/Los_Angeles /etc/localtime`
- `refind-install`
- install inkscape, nodejs, npm (could have done as part of pacstrap but w/e)
- set hostname to 'stushiba' in /etc/hostname and /etc/hosts
- install iw, wpa_supplicant, and dialog in hopes that I'll be able to use my wifi
- `passwd`
- `EDITOR=nano visudo`, uncomment the first wheel line
- `useradd -c "Stuart P. Bentley" -m -G wheel stuart`
- `passwd stuart`
- `exit`

reboot after leaving chroot

## rebooting

try starting arch, get a bunch of errors, remember there's a thing to do, cut system power and retry

reboot, F12, select rEFInd, jam edit keys until seeing the option for editing launch options

add `iommu=pt` to kernel opts because for some reason everything USB fails without it

## after booting and logging in

### hilarious comedy of errors

- `curl aur.sh`
- oh wait I have no dhcp
- crap what's the thing to start dhcpcd with systemctl
- `sudo pacman -S bash-completion`
- derp, can't reach remotes to install without an ip
- `sudo systemctl start dhcpcd.service`
- `curl aur.sh`
- hmm still doesn't seem to be working
- `if link`
- derp
- `ip link`
- `sudo systemctl start dhcpcd@enp3s0.service`
- what it says there was a problem
- `sudo systemctl status dhcpcd@enp3s0.service`
- wait I never enabled dhcp that's not a thing
- derp
- `curl aur.sh`
- works now, I guess that first `systemctl start dhcpcd.service` must have finished
- ¯\\\_(ツ)_/¯

### painful attempts to install packer

I forgot `<(curl aur.sh)` is a thing and that the install flag to makepkg is `-i` and not `-u`
and that bash won't pass scripts flags when piped and basically kept pieing myself in the face

the real punchline comes about half an hour later

- `mkdir aur`
- `cd aur`
- `curl aur.sh | bash -su packer`
- `pacman -S jshon expac`
- `sudo install pacman -S jshon expac`
- `curl aur.sh | bash -su packer`
- `packer -S rtl8812au-dkms-git`
- `ls`
- `man makepkg`
- `curl aur.sh | bash -i packer`
- `curl aur.sh | bash -is packer`
- `curl aur.sh | bash -ifs packer`
- `rm -rf packer`
- `curl aur.sh | bash -ifs packer`
- `sudo packer -S rtl8812au-dkms-git`
- packer is not installed
- `sudo curl aur.sh | bash -ifs packer`
- `rm -rf packer`
- `sudo curl aur.sh | bash -ifs packer`
- `sudo packer -S rtl8812au-dkms-git`
- packer is still not installed
- `man pacman`
- `ls packer`
- `sudo pacman -S packer/packer-20150808-1-any.pkg.tar.xz`
- `sudo pacman -U packer/packer-20150808-1-any.pkg.tar.xz`
- `sudo packer -S rtl8812au-dkms-git`
- error involving `--asroot` not being an option to `makepkg`
- `man makepkg`
- `sudo packer -S rtl8812au-dkms-git`
- not seeing a problem in the pkgbuild or anything
- still won't work
- furious googling
- https://bbs.archlinux.org/viewtopic.php?id=191793
- reading the guy being a dick about `sudo`
- reads more
- oh wait

### packer is finally installed and I know how to use it

still dont know how to dkms tho

- `packer -S rtl8812au-dkms-git`
- barf about not having the kernel headers
- frantic searching and headscratching trying to remember what the name of the package with the Linux headers is
- `pacman -S linux-headers`
- `sudo pacman -S linux-headers`
- installing the headers triggers DKMS and builds the wifi driver

## and now we're caught up: opening https://gist.github.com on my Chromebook

at this point I remember that I should be writing this down (always log tweaks you make to a pet box, especially if you're
breaking the package database with compiled kernel modules):

- write section one from memory `pacman -Qq | less` for what I might have pacstrapped, and `head /etc/pacman.d/mirrorlist` for the reflector command
- write the second part from the output of `cat /root/.bash_history` like I said
- `less ~/.bash_history`

contents of `/home/stuart/.bash_history`:

```
#!/bin/bash
d=${BUILDDIR:-$PWD}
for p in ${@##-*}; do cd "$d"; curl "https://aur.archlinux.org/cgit/aur.git/snapshot/$p.tar.gz" |tar xz; cd "$p"; makepkg ${@##[^\-]*}; done
#!/bin/bash
d=${BUILDDIR:-$PWD}
for p in ${@##-*}; do cd "$d"; curl "https://aur.archlinux.org/cgit/aur.git/snapshot/$p.tar.gz" |tar xz; cd "$p"; makepkg ${@##[^\-]*}; done
#!/bin/bash
d=${BUILDDIR:-$PWD}
for p in ${@##-*}; do cd "$d"; curl "https://aur.archlinux.org/cgit/aur.git/snapshot/$p.tar.gz" |tar xz; cd "$p"; makepkg ${@##[^\-]*}; done
#!/bin/bash
d=${BUILDDIR:-$PWD}
for p in ${@##-*}; do cd "$d"; curl "https://aur.archlinux.org/cgit/aur.git/snapshot/$p.tar.gz" |tar xz; cd "$p"; makepkg ${@##[^\-]*}; done
#!/bin/bash
d=${BUILDDIR:-$PWD}
for p in ${@##-*}; do cd "$d"; curl "https://aur.archlinux.org/cgit/aur.git/snapshot/$p.tar.gz" |tar xz; cd "$p"; makepkg ${@##[^\-]*}; done
```

- using arrow keys, write the last part from the terminal's live command history buffer

## changes after creating build log

- yet to see if wifi module actually works
- https://wiki.archlinux.org/index.php/General_recommendations
- decide not to do any of https://wiki.archlinux.org/index.php/Activating_Numlock_on_Bootup in favor of just seeing if the bios has something
- remember display managers are a thing
- install and enable lightdm
- discover deepin and figure what the heck, install deepin and deepin-extra too
- Add iommu=pt to boot configs in /boot/refind_linux.conf
- /boot/EFI/refind/refind.conf:
  - change `scanfor` to be the Mac default (incorporating BIOS boot options)

### reboot

- "Incorrect password" every time at lightdm
- Nothing anybody is saying online helps
- open another teminal and change password to "lol"
- try again
- works
- note to self that I should file a bug about this
- enabled and started the NetworkManager.service
- uninstalled chromium, installed google-chrome (dat netflix)
- installed a bunch of WM/DEs, text editors, and goofs:
  - scite
  - scribes
  - tea
  - gobby
  - mate, mate-extra
  - lesstif
  - windowmaker
  - notion
  - i3-wm
  - xfce4
  - xfce4-goodies
  - bsd-games
  - fortune-mod
  - cowsay
  - ponysay
  - sl
- `sudo pacman -S $(pacman -Ssq adobe pro-fonts)`
- installed some driver stuff:
  - mesa-vdpau
  - libva-vdpau-driver
- installed obs-studio

## updated 2016-12-11

booted back into Linux because Steam on Windows is being horrible (1-byte-per-second disk write times) and Windows Update is hung and I think I just need to let that disk cool off a bit and I should check out what's new Linux-side.

I haven't touched this since probably September (going by my Tabalanches), so I ran a BIG HONKING STINKING HUGE 500+ PACKAGE `packer -Syu`, it required some geoclue magic and the ttf-dejavu force that's listed on the Arch frontpage right now.

got everything updated and installed some stuff (that new package that makes steam native runtime work correctly, *finally* installed `bash-completion`), rebooted.

wifi adapter kernel module is somehow broken again roflofloflasdgf;;;;;, went over and grabbed my ethernet cable and plugged in

searching for what's up (googling "rtl8812 arch")... not really sure but I think I might pivot on over to https://aur.archlinux.org/packages/rtl8812au_rtl8821au-dkms-git/ which looks a lot more actively maintained

No amount of `packer -S rtl8812au-dkms-git` has resurrected the old driver so I'm moving on over to gnab's

when I update/remove it it keeps saying there are no instances of it in the dkms tree which is weird:

```
:: Running pre-transaction hooks...
(1/1) Remove DKMS modules
==> dkms remove rtl8812au/r62.be75251 -k 4.8.13-1-ARCH
:: Processing package changes...
Error! There are no instances of module: rtl8812au
r62.be75251 located in the DKMS tree.
error: command failed to execute correctly
(1/1) removing rtl8812au-dkms-git                  [######################] 100%
```

probably because it removes it first, idk

moving to rtl8812au_rtl8821au-dkms-git fixed it so whatever

## 2017-05-03

some finagling over https://www.archlinux.org/news/ca-certificates-utils-20170307-1-upgrade-requires-manual-intervention/

pacman complained about gstreamer0.10-good-plugins conflicting with something, put a pin in that and ran the suggested commands

I tried the aforementioned procedure but apparently messed it up, solved by re-running `sudo pacman -Su ca-certificates-utils` after the full upgrade

reran `reflector --verbose -c US -f 200 --sort score -p https --save /etc/pacman.d/mirrorlist` because [the current configuration's primary mirror has some kind of certificate problem](https://www.archlinux.org/mirrors/localmsp.org/714/)

returning to the aforementioned issues with gstreamer, there were some weird issues around trying to remove it (possibly because I hadn't updated yet or something), anyway, after updating I ran `sudo pacman -Rs $(pacman -Qdtq)` to clear out the cruft of whatever'd been replaced in the past, this removed:

```
Packages (88) dleyna-connector-dbus-0.2.0-2  dleyna-core-0.5.0-1  faac-1.28-5
              gnome-mime-data-2.18.0-7  gstreamer0.10-bad-0.10.23-20
              gstreamer0.10-ugly-0.10.19-16  gupnp-av-0.12.10-1
              gupnp-dlna-0.10.5-2  js17-17.0.0-4  json-c-0.12.1-1
              l-smash-2.9.1-1  ladspa-1.13-6  libakonadi-qt4-1.13.0-12
              libcdaudio-0.99.12.p2-1  libdmtx-0.7.4-5  libidl2-0.8.14-4
              liblrdf-0.5.0-3  libmp4v2-2.0.0-4  libyaml-0.1.7-1
              musicbrainz-2.1.5-7  net6-1.3.14-5  orbit2-2.14.19-3
              perl-ipc-system-simple-1.25-1  perl-xml-parser-2.44-3
              python-appdirs-1.4.3-1  python-beaker-1.8.1-2
              python-mako-1.0.6-2  python-markupsafe-1.0-1
              python-packaging-16.8-2  python-pyparsing-2.2.0-1
              python-setuptools-1:35.0.2-1  python2-cssselect-1.0.1-1
              python2-lxml-3.7.3-1  python2-mpd-0.3.0-5  qjson-0.9.0-1
              raptor-2.0.15-6  soundtouch-1.9.2-1  spandsp-0.0.6-1
              wildmidi-0.4.0-1  xapian-core-1:1.4.2-1  cddb-py-1.4-7
              clutter-gst2-2.0.18-1  cython-0.25.2-2
              deepin-dbus-factory-3.1.4-1  deepin-file-manager-backend-0.1.16-4
              deepin-nautilus-properties-3.14.3-1  dleyna-renderer-0.5.0-1
              elfutils-0.168-1  faenza-icon-theme-1.3.1-4
              flattr-icon-theme-1.3-2  gnome-vfs-2.24.4-10
              gobject-introspection-1.52.1-1  gperf-3.1-1
              gstreamer0.10-bad-plugins-0.10.23-20
              gstreamer0.10-python-0.10.22-2
              gstreamer0.10-ugly-plugins-0.10.19-16  intltool-0.51.0-2
              kdepimlibs4-4.14.10-11  lcms-1.19-5  lib32-jansson-2.9-1
              lib32-js17-17.0.0-6  lib32-json-c-0.12.1-1  libbonobo-2.32.1-4
              libgtksourceviewmm2-2.10.3-2  libkactivities4-4.13.3-3
              libxml++-3.0.1+4+g2af973f-2  metacity-3.24.1-1  mutagen-1.37-1
              obby-0.4.8-3  perl-file-basedir-0.07-1  prison-5.33.0-1
              python2-atspi-2.24.0-1  python2-bottle-0.12.13-1
              python2-crypto-2.6.1-5  python2-deepin-vte-0.28.2.20130904-1
              python2-keybinder2-0.3.1-1  python2-pyasn1-0.2.3-1
              python2-pyquery-1.2.17-2  qca-qt4-2.1.3-3  qtwebkit-2.3.4-4
              ruby-2.4.1-3  sonata-1.6.2.1-13  strigi-0.7.8-9  vte-0.28.2-8
              webkitgtk-2.4.11-6  x264-2:148.20170121-1  xdotool-3.20160805.1-1
              zeitgeist-1.0+1+g1bcc8585-1
```

still some stuff I wasn't sure I wanted to ditch riding along here, so I looked into it and found that I'd missed [a brief window where xfce4-mixer had been replaced by xfce4-pulseaudio-plugin](https://bugs.archlinux.org/task/52686), so I installed that and uninstalled xfce4-mixer, and then `sudo pacman -Rsc gstreamer0.10` was basically fine

## 2017-05-06

Somehow I only installed SSH (as in the `openssh` package) **today**, whaaaaaaaa

I am just **stunned** I hadn't done that yet. I kinda thought it was part of the Arch base install, but I guess that's just libssh

I might generate some keys for this so I can get on GitHub, but I don't know, I'm kinda tired, maybe tomorrow

## 2017-05-07

I got kind of fed up with emoji and CJK characters not displaying, so I did a `sudo pacman -S --needed noto-fonts noto-fonts emoji noto-fonts-cjk` (turns out they were *all* yet-to-be-installed so the `needed` was superfluous), and then the system hung for *minutes* (I went downstairs and did laundry, so I don't know how many minutes, but it was probably somewhere between 5 and 15) - popping over to tty3 or whatever (ctrl-alt-f3) and running `top` showed that it was children of `gnome-session-b` (like `gsd-xrandr`, `gsd-color` etc) getting pegged, one after the other (probably propagating the font updates).

Anyway I think I'll do a `ssh-keygen -t rsa -b 4096` now
