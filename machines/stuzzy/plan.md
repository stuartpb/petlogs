- add the thing so bash completion is case insensitive
- NodeMCU stuff

## GNOME issues

- turn off the bell sound when at the end of an input box and hitting the arrow key trying to go past it
- change the default color scheme that applies to Terminal to be dark?
- make it so gsudo doesn't prompt me for my password, just like sudo?
  - might need to switch to pkexec: https://unix.stackexchange.com/questions/174214/is-there-a-graphical-sudo-kdesudo-gksudo-su-to-root-tool-that-works-pas/174249
- upstream touchpad patch in gtk+
- what is the deal with these ligatures in monospaced text? it's happening in Google Chrome, and it sucks.
- Someone needs to tell GNOME "Disable until Tomorrow" for a program that goes from sunrise until sunset (Night Light) should use a definition of "tomorrow" that isn't based around 12:00 AM :facepalm:
- Could have the decency to bring a window I'm trying to drag a file into to the front after a second, like maybe rearrange the windows a little or just make it so it's not too tough to refocus on the original window if you need to drag back to the original (ie. if the whole original window was within the bounding box of the target, move the window to the edge nearest where the cursor entered, off by like half the icon size or whatever), and returning to the original returns focus immediately so you don't end up in the "oh I've made a mess of these" state

## Atom tweaks

- Is there an option to always Reveal in Tree View whenever you switch to a buffer for a file that's within your project? Should this be a package? Is there already one?

## grand plan

these are tasks that involve more than just stuzzy, but the end goal is to have OS partitions for stuzzy, so I'm listing it all here

- [x] put ateworld back on the desktop
- [x] move archive from boatoar to ateworld
- [x] write windows 10 install media to cruzer drive
- [x] bootstrap sturling mk iii to c720p's old storage
- [x] reboot stutendo into sturling to rescue stushiba
  - arch chroot
  - remove AUR package for wifi adapter (since I'm cabled again)
  - pacman -Syu
- [x] reboot into stushiba
  - [x] image stuzate drive directly to ateworld (don't gzip)
    - since I'd like to be able to mount it
    - and compression would only save like 1/32 the drive's capacity
    - and not having to depend on squashfs for anything is worth at *least* that
    - not to mention that this will presumably be much faster
  - [ ] unmount the stuzate drive and swap with the old storage from this machine, chassis to chassis
    - this means my desktop will have 16gb of flash storage more or less mounted within it for basically no reason, which I am fine with
  - [ ] reboot stuzzy with Windows 10 install media and stuzate drive plugged in
