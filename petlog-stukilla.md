uuuuuuugh

installed cgpt on sd card which is whatever alias

changed sdcard hostname so I'd know when I'm experimenting on that and not the other one

installed libgl-mesa because the other 4 are for some kind of special chip that's irrelevant

installed the libinput whatever instead of the evdev one

installed the regular libx264 because 10bit is mostly an encoding advantage and doesn't have hardware support

went with noto fonts for ttf-font satisfaction because hey why not that's chromey (obvs I'd rather do them all but it only prompted for one)

install git, metasploit, chromium, awesome, notion, budgie-desktop, lightdm

bunch of other stuff that's probably in paclog

bashed my head against the wall for a while because I didn't consult notes or wiki about having to install a greeter for lightdm, and didn't think to check the lightdm log file to figure this out

uncommented the en us utf8 locale line in /etc/localegen.conf (because apparently alarm does everything else but not THAT) then ran localegen

uninstalled xf86-input-evdev and -synaptics which got inadvertently installed while trying to make lightdm work

## todo

- make stuart user who is wheel
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

## to answer

- is there any way to speed this up a little? maybe make the video a little less sluggish? hardware acceleration?
- what was that one mesa package that was coming up no results? mesa-vdpau?
- why is packer prompting alarm for password when wheel is set to allow without prompt?