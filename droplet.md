# stuvern (Digital Ocean droplet) changelog
- after sshing in to root with the password emailed to me, I ran `pacman -Syu` and, when that failed, followed the directions at https://www.archlinux.org/news/binaries-move-to-usrbin-requiring-update-intervention/ because the starting Arch image was from May
- `useradd -m -g users -G wheel -s /bin/bash stuart; chfn stuart; passwd stuart`
- `EDITOR=nano visudo`, uncomment `%wheel ALL=(ALL) ALL`
