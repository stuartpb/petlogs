# stuquart setup plan

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
