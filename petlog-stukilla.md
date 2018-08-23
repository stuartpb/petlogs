---
tags: [chromebook, x64, archlinux]
---

# Arch Linux on an x64 Chromebook

My mom's Acer C710 reached End of Support a couple months ago, so we went out and got her a new one, and I asked to keep the old one, so that I could have a Chromebook I could reflash to run Linux that *wasn't* a mess of unsupported proprietary chipsets like the old ARM Samsung Chromebook I'd tried to do this with before.

## Right after getting the Chromebook

Per the [upstream directions](https://www.chromium.org/chromium-os/developer-information-for-chrome-os-devices/acer-c7-chromebook), I Powerwashed the machine into Developer Mode by holding Escape and F3 (Refresh) while pressing the power button, then pressing CTRL+D.

Once I was in Developer Mode, I selected "Enable debugging features", then struggled with confusion after booting into CrOS and hitting Ctrl+Alt+T to get a Terminal tab, typing `shell` and then `sudo crossystem dev_boot_usb=1` only to find that the password I'd set when enabling said debugging features wasn't accepted.

After powerwashing again (via the Chrome settings button for it) and setting that password again, I searched around and caught that the password is actually set **directly on the `root` account**, which you have to **log into via vt2 (Ctrl+Alt+F2)** to change the password for `chronos` for sudo at the shell.

Anyway, I logged into vt2, and at that point I'd already downloaded the [MrChromebox Firmware Utility Script](https://mrchromebox.tech/#fwscript), so I just ran it as root via something like `bash /home/chronos/user/Downloads/firmware-util.sh`.

I backed up the firmware to a flash drive I keep around for exactly this kind of hackery (0xdeadcab1e), flashed the lastest Arch Linux installation media to the flash drive I keep dedicated to this exact purpose (my "rescue penguin"), and rebooted by holding F3 (Refresh) and hitting the power button.

## Installing Arch Linux

Running down the [Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide):

`lsblk` to confirm the onboard flash is /dev/sda

firing up `parted`

doing `rm 1` through `rm 11` to remove all partitions that aren't the EFI system partition

realizing that moving a fat16 partition in parted [hasn't been possible for years](https://superuser.com/a/499804), *aaaaaargh*

I've still got an EFI partition just sitting here in the middle of my partition table looking like an idiot
