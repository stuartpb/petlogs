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

## to try

- cinnamon. what's the perf like? better/same as budgie?
- gnome and/or kde/plasma - it looks like I've got the space so I might as well risk it

## todo

- **at least change the root password my god** (if it's safe it should get disabled altogether)
- add stuart to wireshark group (per https://wiki.archlinux.org/index.php/Wireshark)
  - `usermod -Ga wireshark stuart`?
  - why is the wireshark group missing, need to reread that page
- .face, GECOS
- continue fixing audio for headphones maybe
  - write console-command-based instructions for fixing audio?
  - also maybe this should be fixed upstream like hey just maybe?
- remove password protection circa gnome keyring
- do the touch sensitivity thing from the alarm wiki page
- keep plugging away at that upstream kernel stuff
- rewrite the mess that is the current documentation for doing this, link to Google upstream docs
- fix touchpad having a "right button" region (setting)
  - cinnamon's settings panel is no good for this, looks like it's gonna have to be a conf file
  - that'll work for now (done), but the other options via these GUIs don't make a conf file. how do they do their thing? is there a way I can hack `"ClickMethod" "fingers"` (https://wiki.archlinux.org/index.php/Libinput#Common_options) into whatever it is that they rather use?
  - see above notes about how this is apparently some kind of DE-specific thing, and if I want something that'll carry over to other ones, I'll need to pop it in Xorg, so whatever (though also it seems like I'd have to learn that approach ANYWAY since Budgie didn't take the changes in the xorg conf). Though it'd still be cool to have this somewhere in .config instead of /etc maybe
- try non-verified firmware bootstrap chaining
- crack open and remove read-only ring
- flash nv-uboot if it's safe
- maybe also try that second one, with the framebuffer etc?
- fix the dang gnome footprint icon not being the nine-squares icon in budgie
  - see https://github.com/budgie-desktop/budgie-desktop/issues/250
  - oughta post that in https://bbs.archlinux.org/viewtopic.php?id=209293
  - where's the panel/applets/budgie-menu/com.solus-project.budgie-menu.gschema.xml he mentioned patching installed to, is it world-writable / not package managed
  - also if this was fixed upstream why is it like this here
- either enable netmanager or get a tray icon for netctl

## asked questions

### why is packer prompting me for a password to sudo something when `sudo -l` says I have NOPASSWD permissions?

this should get posted on stackoverflow, but essentially: it's probably because sudo is being evaluated in the context of `root`, and not the original user, and `root` isn't in `wheel`, and as such matches the `root ALL=(ALL) ALL` rule, which doesn't have nopasswd. not sure if this is worth fixing, I mean, probably? prompting root for their own password when they *already have root permissions* seems a little silly (is there a way to tell sudo "any user is allowed to sudo themselves without a password"?)

no, wait, that is, in fact, supposed to be the default behavior (see `man sudoers` section on User Authentication). so now, uh, I don't know? this feels like a bug: it reads SUDO_USER to determine that the original invoker is not root, but then it compares based on the rules that would apply for root (not the invoking user).

### what's the deal with this "aur" repository?

apparently this? https://github.com/archlinuxarm/PKGBUILDs/tree/master/aur

## to answer

- is there any way to speed this up a little? maybe make the video a little less sluggish? hardware acceleration?
- should I install xf86-video-armsoc-chromium? is there more graphics driver coverage I'm missing here?
- what's up with mesa-vdpau being AUR only here?
- why are hangouts windows not having the hangouts icon in the budgie panel? what's it going to take to fix this?
- why does the panel keep disappearing when I (accidetally) hit alt+F2?
