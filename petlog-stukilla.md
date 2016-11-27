# installing arch linux on a chromebook

## uuuuuuugh

followed the pretty straightforward instructions to put the alarm image on an sd card from chrome

tried to follow the instructions again to wipe chromium etc but, oops, no cgpt within Arch

installed cgpt on sd card via `pacman -s cgpt` (which is an alias for `vboot-utils` somehow?), lucky break

changed sdcard hostname so I'd know when I'm experimenting on that and not the locally-installed system

installed libgl-mesa because the other 4 are for some kind of special chip that's irrelevant

installed the libinput whatever instead of the evdev one

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

changing password to something new, too

## todo

- add stuart to wireshark group (per https://wiki.archlinux.org/index.php/Wireshark)
- .face
- hostname
- fix audio
- do the touch sensitivity thing
- try mainline upstream kernel (via sd card), see if all hardware works with new kernel
- rewrite the mess that is the current documentation for doing this, link to Google upstream docs
- fix touchpad having a "right button" region (setting)
- try non-verified firmware bootstrap chaining
- crack open and remove read-only ring
- flash nv-uboot if it's safe
- maybe also try that second one, with the framebuffer etc?
- fix the dang gnome footprint icon not being the nine-squares icon in budgie
- either enable netmanager or get a tray icon for netctl

## asked questions

### why is packer prompting me for a password to sudo something when `sudo -l` says I have NOPASSWD permissions?

this should get posted on stackoverflow, but essentially: it's probably because sudo is being evaluated in the context of `root`, and not the original user, and `root` isn't in `wheel`, and as such matches the `root ALL=(ALL) ALL` rule, which doesn't have nopasswd. not sure if this is worth fixing, I mean, probably? prompting root for their own password when they *already have root permissions* seems a little silly (is there a way to tell sudo "any user is allowed to sudo themselves without a password"?)
`

## to answer

- is there any way to speed this up a little? maybe make the video a little less sluggish? hardware acceleration?
- what's up with mesa-vdpau being AUR only here?
- why is packer prompting alarm for password when wheel is set to allow without prompt?
- why are hangouts windows not having the hangouts icon in the budgie panel? what's it going to take to fix this?
- what's this "aur" repository?