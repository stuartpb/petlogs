---
tags: [chromebook, x64, archlinux]
---

# Arch Linux on an x64 Chromebook

My mom's Acer C710 reached End of Support a couple months ago, so we went out and got her a new one, and I asked to keep the old one, so that I could have a Chromebook I could reflash to run Linux that *wasn't* a mess of unsupported proprietary chipsets like the old ARM Samsung Chromebook I'd tried to do this with before.

## 2018-08-22

### Right after getting the Chromebook

Per the [upstream directions](https://www.chromium.org/chromium-os/developer-information-for-chrome-os-devices/acer-c7-chromebook), I Powerwashed the machine into Developer Mode by holding Escape and F3 (Refresh) while pressing the power button, then pressing CTRL+D.

Once I was in Developer Mode, I selected "Enable debugging features", then struggled with confusion after booting into CrOS and hitting Ctrl+Alt+T to get a Terminal tab, typing `shell` and then `sudo crossystem dev_boot_usb=1` only to find that the password I'd set when enabling said debugging features wasn't accepted.

After powerwashing again (via the Chrome settings button for it) and setting that password again, I searched around and caught that the password is actually set **directly on the `root` account**, which you have to **log into via vt2 (Ctrl+Alt+F2)** to change the password for `chronos` for sudo at the shell.

Anyway, I logged into vt2, and at that point I'd already downloaded the [MrChromebox Firmware Utility Script](https://mrchromebox.tech/#fwscript), so I just ran it as root via something like `bash /home/chronos/user/Downloads/firmware-util.sh`.

I backed up the firmware to a flash drive I keep around for exactly this kind of hackery (0xdeadcab1e), flashed the lastest Arch Linux installation media to the flash drive I keep dedicated to this exact purpose (my "rescue penguin"), and rebooted by holding F3 (Refresh) and hitting the power button.

### Installing Arch Linux

Running down the [Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide):

`lsblk` to confirm the onboard flash is /dev/sda

firing up `parted`

doing `rm 1` through `rm 11` to remove all partitions that aren't the EFI system partition

realizing that moving a fat16 partition in parted [hasn't been possible for years](https://superuser.com/a/499804), *aaaaaargh*

I've still got an EFI partition just sitting here in the middle of my partition table looking like an idiot

### let's get ddangerous

going into `fdisk` and doing `n 1 2048 +32767 t 1 1 w` to create a new EFI System partition at the first available aligned sector (the default when there are no partitions), of the same size as the existing one (32768 sectors)

ran `dd conv=notrunc if=/dev/sda12 of=/dev/sda1 bs=1M` to copy the partition, then checked that the copy worked with `mount /dev/sda1 /mnt && ls /mnt && umount /mnt`

`fdisk` again, hitting `x` to get into Expert Mode

`n 1` to give the new partition the name `EFI-SYSTEM` just like the old one

not copying the old UUID because why bother

returning to basic mode with `r`

doing `d 12` to delete the old EFI-SYSTEM partition that was in a weird place

hitting `n` and just enter for the rest of the defaults to create an appropriate Linux partition out of the rest of the disk

momentarily questioning if all this partition mucking about is worth it if I'm just going to order a cheap SSD to replace the stock flash anyway

running `mkfs.ext4 /dev/sda2` (if I do end up buying a new SSD, I'll probably redo this with F2FS or btrfs, probably the latter because I'm obsessed with features like COW)

only at this point, realizing that pacstrap can't use the local package store, do I run `wifi-menu`

gonna pacstrap a dev environment with a (hopefully) nice wm as `pacstrap /mnt base base-devel budgie`

### rote following of the guide

running `genfstab -U /mnt >> /mnt/etc/fstab` and `arch-chroot /mnt` as perscribed in the guide

running `ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime` even though honestly I wish systems didn't have a concept of "local time" beyond the UI but whatever

running `hwclock --systohc` even though it's not clear why that's necessary

uncommenting `en_US.UTF-8 UTF-8` in `/etc/locale.gen` and running `locale-gen`

also making an `/etc/locale.conf` with `LANG=en_US.UTF-8` in it

`echo stukilla > /etc/hostname`

honestly, not doing the "add an entry for your hostname as 127.0.1.1 in /etc/hosts" thing, life is too friggin' short

### a little more setup for the GUI stuff

`pacman -S lightdm lightdm-gtk-greeter && systemctl enable lightdm`

### setting myself up

uncommenting the "allow all members of wheel to sudo without a password" line via `EDITOR=nano visudo`

`useradd -c "Stuart P. Bentley" -m -G wheel stuart && passwd stuart`

okay let's see if this thing boots

## 2018-08-23

Well whaddaya know, copying the leftover EFI-SYSTEM partition from Chrome OS doesn't count as establishing a bootloader for the new OS

Looking at https://wiki.archlinux.org/index.php/EFISTUB, I'm going to try adding a boot entry for Arch directly:

`efibootmgr --disk /dev/sda --part 1 --create --label "Arch Linux" --loader /vmlinuz-linux --unicode "root=$(blkid -o export /dev/sda2 | grep '^PARTUUID=') rw initrd=\\initramfs-linux.img" --verbose`

Rebooted, and... still nothing (even after retrying, having originally typed `--part 2` thinking it was supposed to refer to the OS partition). Booting back into rescue media showed that the new boot entry didn't appear to persist, so I'm going to try the process for creating a boot entry directly from UEFI Shell.

Trying to create the entry from UEFI Shell v2 made it clear that the issue was that the EFI partition didn't contain the EFISTUB data, since I'd never mounted it as `/boot`.

Guhhh... looking at what *is* in `/boot`, `initramfs-linux-fallback.img` is 28M. That's too big for the 16M partition I (re)used from the ChromeOS configuration. I'm... pretty much going to have to try this all over again.

### from the top

Going into `fdisk /dev/sda`, deleting the existing partitions, creating a new EFI System partition at 2048 ending at `+256MiB` (the minimum for some drive types as described on [the Arch Wiki](https://wiki.archlinux.org/index.php/EFI_system_partition) - the Wiki recommends dedicating a full 512MiB "to avoid potential problems with some UEFI implementations", but I just don't feel like reserving so much of this limited disk space on such an esoteric concern), and a Linux filesystem partition for the rest. (also giving them the labels of "EFI-SYSTEM" and "onboard-flash")

Formatting the partitions (ESP as FAT32), mounting them (the ESP into `/mnt/boot`), doing `wifi-menu`, and running `pacstrap /mnt base base-devel budgie-desktop lightdm lightdm-gtk-greeter`

Redoing all the stuff above from "rote following of the guide" down (skipping the `pacman -S` for lightdm obviously) and rebooting... yup, still not working. Booting back into the rescue media and going to try rerunning the `efibootmgr` command above.

Ugh, my ESP appears to have nothing in it, and there's no entry for `/boot` in `/etc/fstab`. It looks like I didn't mount /boot, even though I'm almost certain I did. (I might have typo'd it as `mount /dev/sda1 /boot` instead of `mount /dev/sda1 /mnt/boot`, as that's what happens every time I try to do this in recovery.) I'm going to try moving the contents of /boot into that partition and regenerating the fstab, seeing if that works.

### aaaaargh

Okay, I moved everything around, and was finally able to boot from `vmlinuz-linux` using the UEFI Shell on the rescue media, but now I've got a crash on startup around `mei_me 0000:00:16.0: initialization failed.` I don't have time to deal with this right now

## 2018-08-24

It's becoming clear that trying to boot directly via EFISTUB is more trouble than it's worth (if it's even possible with this firmware at all), so I'm just going to chump out and install a bootloader.

Reading the wiki, it seems like the simplest way to do this will be to use [systemd-boot](https://wiki.archlinux.org/index.php/Systemd-boot), so that's what I'm doing: booting back into rescue media, applying the mounts, `arch-chroot`ing into `/mnt`, and running `bootctl --path=/boot install`.

Hmm, for some reason the only option that appears now is "Reboot Into Firmware Interface".

## 2018-08-29

Okay, now that I look at this a little more closely, apparently you're supposed to [configure](https://wiki.archlinux.org/index.php/systemd-boot#Configuration) systemd-boot after installing it, and the fact that I didn't do that is probably why this isn't working.

I'm editing `/boot/loader/loader.conf`, and for some reason, the last line of the file is `default 4561e26e9f224e4aa22ef630e03b93b9-*`. After doing an `fgrep -sr 4561e26 /`, it comes to light that this value comes from `/etc/machine-id`, a file I'd never encountered before, documented [in systemd's manual](https://www.freedesktop.org/software/systemd/man/machine-id.html). After reading [the spec](https://github.com/systemd/systemd/blob/master/doc/BOOT_LOADER_SPECIFICATION.md) and thinking about it for a while, I *eventually* understand why this naming convention is in place (so that multiple OS instances, with each having multiple boot options, can be installed to the ESP without conflicts). I would expect that there'd be something built into the distro to handle this as part of its base, but it sounds like the Arch boot process is so fragmented that everybody is basically just rolling their own at this point.

Before I mess with all this, I'm remembering that I ran into problems on 2018-8-23 with the Intel Management Engine module stuff, which might point to the need to include an initrd for Intel microcode as the Arch Wiki keeps imploring. I try to run `wifi-menu`, only for the system to tell me I need to install `dialog`. I exit the chroot and run `wifi-menu` from the outer shell (which has `dialog` and is fine), then `arch-chroot /mnt` again and run `pacman -S --asdeps dialog`.

Now that I'm online and have set up the system to not have me locked out on boot, I run a quick `pacman -Syu` to bring the system back up to date, then move on to get the microcode package with `pacman -S intel-ucode`.

Now that I've got all that, I create a config file by running the following multi-line command:

```
echo -n "title Arch Linux
machine-id $(cat /etc/machine-id)
efi /vmlinuz-linux
options initrd=/intel-ucode.img
options initrd=/initramfs-linux.img
options root=$(blkid -o export /dev/sda2 | grep '^PARTUUID=')
" > /boot/loader/entries/$(cat /etc/machine-id)-arch-local-default.conf
```

Having finished with all this, I exit the chroot and reboot. The loader appears to work properly, and I get the same problem I was experiencing before: Light Display Manager fails to start, and the `mei_me` module fails initialization.

To diagnose what's going wrong with lightdm, I boot back into the rescue media: I note that the `mei_me` issue is apparent in startup there, too, and is probably unrelated.

Looking at `/var/log/lightdm/lightdm.log`, it looks like it wasn't able to start an X server - I've probably got more packages I need to install. `wifi-menu` still doesn't work because `wpa_supplicant` isn't installed (d'oh), so I go back out of the chroot and redo `wifi-menu` there, re-chroot, `pacman -S wpa_supplicant`, and check that `wifi-menu` works now (it does).

[This thread](https://bbs.archlinux.org/viewtopic.php?id=235006) reveals that, for some reason that is beyond me, desktop environment packages like `budgie-desktop` don't include `xorg-server` as a dependency, which is apparently required by lightdm / the GTK greeter. (Don't ask me why neither of *those* packages list xorg-server as a dependency, either.)

On rebooting, everything *finally* works - just in time for me to order a replacement storage device for this machine and have to do all this over again.
