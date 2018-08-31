## GNOME issues

- turn off the bell sound when at the end of an input box and hitting the arrow key trying to go past it
- change the default color scheme that applies to Terminal to be dark?
- make it so gsudo doesn't prompt me for my password, just like sudo?
  - might need to switch to pkexec: https://unix.stackexchange.com/questions/174214/is-there-a-graphical-sudo-kdesudo-gksudo-su-to-root-tool-that-works-pas/174249
- upstream touchpad patch in gtk+

## grand plan

these are tasks that involve more than just stuzzy, but the end goal is to have OS partitions for stuzzy, so I'm listing it all here

- [x] put ateworld back on the desktop
- [x] move archive from boatoar to ateworld
- [x] write windows 10 install media to cruzer drive
- [ ] bootstrap sturling mk iii to c720p's old storage
- [ ] reboot stutendo into sturling to rescue stushiba
  - arch chroot
  - remove AUR package for wifi adapter (since I'm cabled again)
  - pacman -Syu
- [ ] reboot into stushiba
  - [ ] image stuzate drive directly to ateworld (don't gzip)
    - since I'd like to be able to mount it
    - and compression would only save like 1/32 the drive's capacity
    - and not having to depend on squashfs for anything is worth at *least* that
    - not to mention that this will presumably be much faster
  - [ ] unmount the stuzate drive and swap with the old storage from this machine, chassis to chassis
    - this means my desktop will have 16gb of flash storage more or less mounted within it for basically no reason, which I am fine with
  - [ ] reboot stuzzy with Windows 10 install media and stuzate drive plugged in
