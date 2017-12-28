---
tags: [chromebook, arm, archlinux]
---

# installing arch linux on a chromebook

## uuuuuuugh

followed the pretty straightforward instructions to put the alarm image on an sd card from chrome

tried to follow the instructions again to wipe chromium etc but, oops, no cgpt within Arch

installed cgpt on sd card via `pacman -s cgpt` (which is an alias for `vboot-utils` somehow?), lucky break

changed sdcard hostname so I'd know when I'm experimenting on that and not the locally-installed system

installed mesa-libgl because the other 4 (imx-gpu-viv) are for some kind of special chip that's irrelevant, and it's kind of annoying that I was asked, actually

installed the xf86-input-libinput instead of the evdev one

installed the regular libx264 because 10bit is mostly an encoding advantage and doesn't have hardware support

went with noto fonts for ttf-font satisfaction because hey why not that's chromey (obvs I'd rather do them all but it only prompted for one)

install git, metasploit, chromium, awesome, notion, budgie-desktop, lightdm

bunch of other stuff that's probably in paclog

bashed my head against the wall for a while because I didn't consult notes or wiki about having to install a greeter for lightdm, and didn't think to check the lightdm log file to figure this out

uncommented the en us utf8 locale line in /etc/localegen.conf (because apparently alarm does everything else but not THAT) then ran localegen

uninstalled xf86-input-evdev and -synaptics which got inadvertently installed while trying to make lightdm work

## running

installed guake, changed the hotkey to F1 because lol chromebooks have no f12

logging out and running `sudo usermod -l stuart -m -d /home/stuart alarm && groupmod -n stuart alarm` because why keep a dormant user around, especially when I've already started to customize this one

changing password to something new, too (this broke the gnome keyring so now that's something I need to fix)

created a conf file to make the touchpad finger-friendlier, but still not wild about it (see todo below)

## adventures in kernel fuckery

popped sdcard back in and ran the kernel update stuff

booted and it seems to work but yeah a bunch of stuff is broken

tried to get some desktop environments installed but didn't have much luck

touchpad was broken after doing lightdm -> ldxe, then after rolling back the kernel it seemed fine so it looks like the kernel probably breaks touchpads right now

## uuuuuuuuuuuuuuuugh

got into another bloody mess after booting into the onboard os because of my attempt to fix the touchpad. Stupid https://wiki.archlinux.org/index.php/Libinput#Common_options doesn't explain that you're supposed to precede the "options" it lists with `Option` (oh yeah let's just assume that I love writing xorg conf files and have the syntax memorized instead of actively trying to figure out how to never use them again), and besides that it's wrong because it's `clickfinger` not `fingers` jfc

anyway that stupid typo made it so the ENTIRE LOGIN MANAGER COULD NOT START so that was fun trying to diagnose based on log files I was seeing no indication of opening and forum threads that render like ass in Lynx/w3m

ok so then I edited the file so that it has that now, but now guess what? IT DOES NOTHING AND I STILL HAVE CLICKS TRIGGERING PASTE IF I PUSH THE WRONG PART OF THE PAD FML

fixing the sound per the instructions on https://archlinuxarm.org/platforms/armv7/samsung/samsung-chromebook, switched the selected card from PulseAudio (instructions didn't mention that bit), arrowed right by like fifty screens until I found these items, which I unmuted (and a few I skipped, marked with [Off]):

- Left Speaker Mixer LINEA
- Left Speaker Mixer LINEB
- Left Speaker Mixer Left DAC
- Left Speaker Mixer MIC1 [Off]
- Left Speaker Mixer MIC2 [Off]
- Left Speaker Mixer Right DAC

a whole bunch more skipping:

- Right Speaker Mixer LINEA
- Right Speaker Mixer LINEB
- Right Speaker Mixer Left DAC
- Right Speaker Mixer MIC1 [Off]
- Right Speaker Mixer MIC2 [Off]
- Right Speaker Mixer Right DAC

AFAICT from testing only one of these (any of them) per speaker needs to be enabled for sound to come out? not sure how it all adds up but whatever

this will probably have to be repeated for headphones

didn't have to do the part where I change the master volume, I think desktop environments take care of that

logged out of budgie because it was driving me nuts and I wanted to try a compositing wm. tried to boot sway and it wouldn't go (kicked back to lightdm). booted to LXDE, and, well, this is interesting:

lxde is way faster - none of the lag I was seeing with Budgie. However, none of the input stuff I set - tap-to-click, natural scrolling - is active, *except* the one I set via Xorg conf (multitouch-only buttons). It's almost as if Budgie wasn't using Xorg at all, but some kind of slower software compositor. Like, this is what I'd expect in some kind of scenario where it's running Weston and Wayland doesn't have a hardware GL implementation. But Budgie apparently has no Wayland support, so that couldn't be it. But I feel like it's probably something like that.

youtube isn't buttery smooth - a little stuttering, tearing - but it's not unwatchable, unlike budgie.

oh yeah, also, this is weird: had to log into github again, and other stuff that suggests cookies in LXDE are handled in a separate sandbox from Budgie (probably GNOME keyring related).

also, Budgie didn't pester me about the keyring at all, not sure if that's just because I didn't do anything that would have triggered it or if it's because the problem magically fixed itself

D'OH, I hit the power button trying to restert the X server with ctrl+alt+backspace. well, upshot is that now I have the fully-reloaded version of my touchpad configuration loaded up. for posterity, `cat /etc/X11/xorg.conf.d/30-touchpad.conf`:

```
Section "InputClass"
  Identifier "libinput touchpad catchall"
  MatchIsTouchpad "on"
  MatchDevicePath "/dev/input/event*"
  Option "Tapping"
  Option "NaturalScrolling"
  Option "ClickMethod" "clickfinger"
  Driver "libinput"
EndSection
```

note the bool options being set to `true` by not specifying a parameter because otherwise I'd have to pick which one word from out of the twelve ways that xorg.conf supports spelling boolean values I want to screw with

`ssh-keygen -t rsa -b 4096` so this machine has its own GH credentials

`passwd -l root`, I feel like I've got things under control enough that I can do all my work from my own account

installed wireshark a while ago, added stuart to wireshark group (per https://wiki.archlinux.org/index.php/Wireshark) via `gpasswd -a stuart wireshark` (I remember having trouble when I tried to do this originally, maybe needed to reboot to reload group info or something)

`sudo chfn -f "Stuart P. Bentley" stuart` because full name data is apparently restricted for some dumb reason

downloaded my github profile image as ~/.face

changed LXDE clock to use `%-I:%M %p` as time format

## 2017-05-23

`sudo pacman -Syu`, which [has caused a problem](https://archlinuxarm.org/forum/viewtopic.php?f=60&t=11670) (see https://github.com/justmoon/node-bignum/issues/98)

installed bluez-utils, enabled bluetooth.service

installed blueman

paired with stuffed elephant

refiddled with all that stuff mentioned around when I originally enabled speakers (somehow they'd re-muted), plus a bunch of of other channels I also unmuted that sounded like they might matter

installed pavucontrol which I really should have done much earlier

also installed bluez-firmware even though I'm pretty sure that doesn't do anything

followed the directions in https://wiki.archlinux.org/index.php/Bluetooth_headset#Headset_via_Bluez5.2FPulseAudio in pavucontrol, reloaded the chrome tab I was experimenting with, and got it to work

installed firefox, festival, espeak, speech-dispatcher, sox, libmad, opusfile

## NEW ENTRIES GO HERE

## to try

- cinnamon. what's the perf like? better/same as budgie?
- gnome and/or kde/plasma - it looks like I've got the space so I might as well risk it

## todo

- remove password protection circa gnome keyring
- keep plugging away at that upstream kernel stuff
- rewrite the mess that is the current documentation for doing this, link to Google upstream docs
- try non-verified firmware bootstrap chaining
- crack open and remove read-only ring
- flash nv-uboot if it's safe
- maybe also try that second one, with the framebuffer etc?
- add emulators and other computer game crap like that I go for every time I set up a Linux system even though this one is supposed to be a dedicated machine for hacking

## quest status

### Bettering Petlogs

- codify what a petlog is and write an explainer of why you should do it every time you install linux (tldr, you're going to be writing a pleading forum post at some point, and having what you did to "cause" your problem is going to go a long way toward making every Linux forum's assholes slightly more shut the hell up)
  - ooh! even better, I should just collect all my petlogs into one repo with a README that has that thing, since I'm not REALLY getting all that much out of Gist for this

is there a way I could use vim or whatever to spawn an editor for this file where, every time I save it, it does a git commit and git push (and optionally prompts me for a commit message and otherwise just pushes it with something like whatever default commit message Gist uses)?

maybe even just a bash script with nano and inotify or something

### Making the Touchpad Work Normally

- fix touchpad having a "right button" region (setting)
  - cinnamon's settings panel is no good for this, looks like it's gonna have to be a conf file
  - that'll work for now (done), but the other options via these GUIs don't make a conf file. how do they do their thing? is there a way I can hack `"ClickMethod" "fingers"` (https://wiki.archlinux.org/index.php/Libinput#Common_options) into whatever it is that they rather use?
  - see above notes about how this is apparently some kind of DE-specific thing, and if I want something that'll carry over to other ones, I'll need to pop it in Xorg, so whatever (though also it seems like I'd have to learn that approach ANYWAY since Budgie didn't take the changes in the xorg conf). Though it'd still be cool to have this somewhere in .config instead of /etc maybe
- make touchpad scrolling smooth like cros instead of a janky wheel-analogue like it is right now?
  - Actually I think the jankiness is just an artifact of lag from the non-compositing draw stuff
- do the touch sensitivity thing from the alarm wiki page
  - pretty sure that's actually a legacy thing from the synaptics input driver that's no longer relevant today

### Managing Wifi

- either enable netmanager or get a tray icon for netctl

I have network-manager-applet installed, as it's a prereq for Cinnamon (and I installed it for Budgie), but I'd rather have systemd take care of bringing up wifi automatically in cooperation with netctl &co, so I haven't activated networkmanager.service.

Right now, I'm in LXDE, and it has a really nice panel widget for network management, and the NetworkManager one just sits there dead next to it, noting that the service isn't running when clicked.

What I want is a way to make it so every DE can have a single applet for networking, and each one using the lightest-weight solution: so LXDE just uses what I've got here, others maybe use that netctl applet, and Cinnamon maybe gets over NetworkManager somehow (like we disable the applet in favor of that other one I described).

### The Power of Power

- **screw with some kind of systemd rule or whatever so that what just happened never happens again** - disable the power button altogether if that's what it takes, but ideally I'd have CrOS's behavior back: tapping the button starts a fade-and-zoom, holding it to completion logs me out (I think they technically ripped this off from OSX but I don't care, it's good)
- lid suspend? (don't think I actually want this generally speaking but it'd be nice to be ready for it)

see https://wiki.archlinux.org/index.php/Chrome_OS_devices#Ignore_using_logind for changing this stuff

I'm doing this now. Just ran a `sudo pacman -Syu` (with the appropriate workaround for that pain-in-the-ass certificates bug thing), then I tried installing `gnome-tweak-tool` and running that, but it ran into this error, probably due to some kind of kernel mismatch or something:

```
Traceback (most recent call last):
  File "/usr/bin/gnome-tweak-tool", line 79, in <module>
    from gtweak.app import GnomeTweakTool
  File "/usr/lib/python2.7/site-packages/gtweak/app.py", line 25, in <module>
    from gtweak.tweakmodel import TweakModel
  File "/usr/lib/python2.7/site-packages/gtweak/tweakmodel.py", line 23, in <module>
    from gtweak.utils import SchemaList, LogoutNotification, Notification
  File "/usr/lib/python2.7/site-packages/gtweak/utils.py", line 20, in <module>
    import tempfile
  File "/usr/lib/python2.7/tempfile.py", line 35, in <module>
    from random import Random as _Random
  File "/usr/lib/python2.7/random.py", line 885, in <module>
    _inst = Random()
  File "/usr/lib/python2.7/random.py", line 97, in __init__
    self.seed(x)
  File "/usr/lib/python2.7/random.py", line 113, in seed
    a = long(_hexlify(_urandom(2500)), 16)
OSError: [Errno 38] Function not implemented
```

so I'm just going to follow the "Ignore using logind" changes

Hmm, trying to restart it, it looks like there's no unit file? WTF?

## 2017-09-19

switched gnome terminal to dark theme

## 2017-10-06

uninstalled packer and installed apacman, which is kept up to date and wouldn't conflict with packer-io, installed apacman-deps, apacman-utils, pkgfile, and rsync

## 2017-11-07: the quest for lid stuff

I want to do some fancy stuff like "just turning the screen off when the lid is closed" (and maybe introducing some kind of audio alert when the system is unplugged), so I'm looking into how to read lid state

stuff like https://unix.stackexchange.com/questions/148197/determine-status-of-laptop-lid suggests there should be a tree to read under /proc/acpi, but there isn't

running `ls -l /usr/lib/modules/$(uname -r)/kernel/drivers/acpi` as mentioned at https://wiki.archlinux.org/index.php/ACPI_modules failed with that entire `acpi` directory not existing

installed evtest based on https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=854388 and I can definitely see the information about the lid when I look at `gpio-keys.7`

welp, this sucks, but it's where I ended up: https://unix.stackexchange.com/questions/244767/sysfs-alternative-to-proc-acpi-button-lid-lid-state/403171#403171

okay, diving into some more heavy nuts and bolts with https://unix.stackexchange.com/questions/265890/is-it-possible-to-get-the-information-for-a-device-tree-using-sys-of-a-running (lol hi @cirosantilli): installing `dtc` and running... well this doesn't tell me much I didn't already know:

```
[stuart@stukilla ~]$ dtc -I fs -O dts /proc/device-tree/gpio-keys/lid-switch/ -f
<stdout>: ERROR (name_properties): "name" property in / is incorrect ("lid-switch" instead of base node name)
Warning: Input tree has errors, output forced
/dts-v1/;

/ {
	name = "lid-switch";
	gpio-key,wakeup;
	debounce-interval = <0x1>;
	linux,code = <0x0>;
	linux,input-type = <0x5>;
	gpios = <0x8 0x5 0x1>;
	label = "Lid";
};
```

the [gpio-keys][dt-docs-1] and [gpio][dt-docs-2] docs don't help me understand much more from this. I thought it might mean something on gpiochip8, possibly gpio13 (gpio8 + 5, the second byte of gpios), but that doesn't appear to reflect the lid state

[dt-docs-1]: https://github.com/torvalds/linux/blob/master/Documentation/devicetree/bindings/input/gpio-keys.txt
[dt-docs-2]: https://github.com/torvalds/linux/blob/master/Documentation/devicetree/bindings/gpio/gpio.txt

I even ran through and retried this for every pin from 8 to 13 via `PIN=8; sudo bash -c "echo $PIN > /sys/class/gpio/export" && cat /sys/class/gpio/gpio$PIN/value && sleep 5 && cat /sys/class/gpio/gpio$PIN/value && sudo bash -c "echo $PIN > /sys/class/gpio/unexport"` etc, closing the lid during the sleep, and no change

OK, so by doing `for pinctrl in /proc/device-tree/pinctrl@*; do dtc -I fs -O dts $pinctrl -f; done | less '+/phandle = <0x8>;' -j.5` I was able to figure out that the device would be labeled gpx3, and by doing `grep -l gpx3 /sys/class/gpio/gpiochip*/label`, I got that the device is gpiochip156, but trying PIN values like 156 and 161 via my test script above, the initial echo to `export` failed with `write error: Device or resource busy`, so I'm guessing there's some kind of exclusive-mode thing there. What a dead end waste of time.

### whatever ugh

Going ahead and installing + enabling acpid:

`sudo pacman -S acpid && sudo systemctl enable acpid && sudo systemctl start acpid`

UGH, it only has an event for when the power key goes down, not up, that's insufficient for what I need

anyway I could go ahead and configure this to set the display brightness to 0 when closed (xsetting dpms is overkill imo), but I'm kind of worn out from all this and am just going to cut it here

## 2017-11-09

since it looks like xf86-video-armsoc-chromium has disappeared from repositories altogether, I uninstalled it in favor of xf86-video-armsoc-git, which still doesn't work (logging into Budgie, it's still excruciatingly slow)

## 2017-12-02

setting up a local npm global setup with `mkdir .npm-global && echo 'PATH=$PATH:$HOME/.npm-global/bin' >> ~/.profile` (this creates .profile for this install)

`.npm-global` being the recommendation of https://docs.npmjs.com/getting-started/fixing-npm-permissions#option-2-change-npms-default-directory-to-another-directory

what I really wanted to do, tbh, was use /usr/local and do everything global through `sudo npm install -g` into /usr/local, but for some reason the default is `/usr` on Arch, and I couldn't get any effect with `sudo bash -c 'prefix = /usr/local >> /etc/npmrc'`, possibly because it says the global config is looked for in `$PREFIX/etc/npmrc` and `$PREFIX` means it's looking in `/usr/etc/npmrc` which is not a thing?

anyway using a non-privileged local dir for my user is fine by me

I was worried about man pages but apparently man actually looks for man pages as a sibling from /bin dirs in `PATH`, which is just wild

`npm config set prefix ~/.npm-global`

having set all this up I ran `npm install -g nodemcu-tool`

## 2017-12-04

installed pip and did `pip install esptool --user`, which created and uses `~/.local`, and makes sense, so I'm redoing the `./npm-global` stuff I set up the other day to use `~/.local` instead

## 2017-12-27

I want to try OctoPrint with my new Qidi Tech I 3D printer, and don't feel like messing with my RPi right now, so I'm using this instead

following the directions from https://github.com/foosel/OctoPrint/wiki/Setup-on-a-Raspberry-Pi-running-Raspbian

Cloned foosel's repo into `git`, then changed my mind and put it directly in my home dir (one less thing to differ from expected)

Installing `python2-virtualenv` because that looks necessary

ran the install script with the venv stuff

did the `mkdir ~/.octoprint` thing

starting OctoPrint and controlling it from my desktop

### octoprint setup

skipping access control (this setup will be local)

skipping connectivity check stuff (let it assume it's online)

sure, enable blacklist whatever

I... guess I'll skip the CuraEngine stuff since I don't have a profile and stuff

setting defaults from a thread on Thingiverse (center origin, build area 227x148x150, top X/Y/Z/E speeds 2500/2500/200/300, 2 extruders)

### stuff not working

installing the GPX plugin in OctoPrint

running `sudo usermod -aG uucp stuart` so I can connect to the serial ports

after doing those I had to open the GPX stuff in settings (failed with no gpx.ini), then log out and back in (reload the user permissions)

got timeouts trying to connect with baud rates above 115200

### working but weird

selected "invert control" for the Z axis in the printer profile because up moved it down and vice versa

## asked questions

### why is my video performance so bad?

- should I install xf86-video-armsoc-chromium? is there more graphics driver coverage I'm missing here?
  - i appear to already have this package, so that's not the problem
- should I maybe replace it with xf86-video-armsoc-git?
- what's up with mesa-vdpau being AUR only here?

Just noticed Guake was translucent over windows in Budgie, only translucent over desktop in LXDE. Score another point for the "Budgie was probably compositing in software" hypothesis.

### what's the deal with this audio situation?

- continue fixing audio for headphones maybe
  - write console-command-based instructions for fixing audio?
  - also maybe this should be fixed upstream like hey just maybe?

just like Managing Wifi, LXDE has a panel widget for volume, but it has a REALLY ugly icon, and right-clicking it and picking '"Volume Control" Settings' brings up alsamixer in LXTerminal, which is REALLY weird. Aren't there better solutions? At least an actual graphical mixer I could use instead of alsamixer? Maybe a dedicated tray icon app (ugh)?

### why is packer prompting me for a password to sudo something when `sudo -l` says I have NOPASSWD permissions?

this should get posted on stackoverflow, but essentially: it's probably because sudo is being evaluated in the context of `root`, and not the original user, and `root` isn't in `wheel`, and as such matches the `root ALL=(ALL) ALL` rule, which doesn't have nopasswd. not sure if this is worth fixing, I mean, probably? prompting root for their own password when they *already have root permissions* seems a little silly (is there a way to tell sudo "any user is allowed to sudo themselves without a password"?)

no, wait, that is, in fact, supposed to be the default behavior (see `man sudoers` section on User Authentication). so now, uh, I don't know? this feels like a bug: it reads SUDO_USER to determine that the original invoker is not root, but then it compares based on the rules that would apply for root (not the invoking user).

### what's the deal with this "aur" repository?

apparently this? https://github.com/archlinuxarm/PKGBUILDs/tree/master/aur

### why do Chromium xtension &c windows have Chromium icons instead of the extension etc's icon?

ie hangouts windows not having the hangouts icon in panel

also: what's it going to take to fix this?

also whenever hangouts notifies, the thing keeps flashing on my main chromium window until I reload whatever tab I had open, ugh this is just as bad as it is on Windows

### why does the budgie panel keep disappearing when I (accidentally) hit alt+F2?

most likely answer based on having used it for three days: because budgie is bad

see also:

- fix the dang gnome footprint icon not being the nine-squares icon in budgie
  - see https://github.com/budgie-desktop/budgie-desktop/issues/250
  - oughta post that in https://bbs.archlinux.org/viewtopic.php?id=209293
  - where's the panel/applets/budgie-menu/com.solus-project.budgie-menu.gschema.xml he mentioned patching installed to, is it world-writable / not package managed
  - also if this was fixed upstream why is it like this here
