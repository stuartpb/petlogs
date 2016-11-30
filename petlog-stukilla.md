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

## asked questions

### why is my video performance so bad?

- should I install xf86-video-armsoc-chromium? is there more graphics driver coverage I'm missing here?
  - i appear to already have this package, so that's not the problem
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
