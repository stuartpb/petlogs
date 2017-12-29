---
tags: [android]
---

# Stuixel, my new Pixel 2 XL

ok so basically with this new phone I've already stated getting pretty wild with it right out of the gate, so I figure I should probably start keeping a petlog of everything I'm doing to it, since it is effectively a pet Linux box in my pocket

## unboxed

I unboxed the phone and all a couple days ago, thinking "oh I'm just going to root this with the stuff in the box, nbd". popped my old micro-SIM in and whoaaa derp, the cable it comes with is USB-C on both ends, and I don't have a freaking MacBook or Chromebook Pixel or Pixelbook or whatever the fuck would make that a reasonable option to adb/fastboot with it

anyway I knew I was just going to blow the whole installation away when I unlocked the bootloader for Magisk and everything so I blew through the initial setup going "skip" as much as I could (even skipping the WiFi setup) so I could enable OEM unlocking and USB debugging and all that stuff there are like a billion tutorials on doing

let it sit on my desk for a few days as I hustled around trying to find an appropriate cable, it was soul-wretching and I don't really want to talk about it

## unlocking stuff

TODO look through my terminal's scrollback and fill this in (see next paragraph)

## actual live setup

I'm writing all this on the phone itself right now and it just hit 15% battery and also it's 3 AM so I'm going to skip the whole pretty-standard setup and Magisk procedure for now (I'll come back and write it up in the morning) to get to the specifics of the ssh identity setup

## setting up ssh

okay so I knew I wanted a Git client and also knew I'd want ConnectBot for SSH, so I installed MGit and the latter via F-Droid and went through a whole bunch of different kerfluffles trying to make an identity in one and import it cleanly into the other (including an arduous `chown` of key files produced in Amaze to MGit's user, which somehow *still* had permission denied, uuuuugh). since afaict there's no way to use Amaze as a file picker in Oreo any more (ie. no way to escape sdcard) and ConnectBot doesn't let you change nicknames after importing and I kind of like ConnectBot's entropy feeder thing, I decided to go with CB producing a 4096-bit RSA key, then had MGit produce its *own* key (naming it `id_rsa`), then opening the files up in Amaze and overwriting the key contents with the keypair from ConnectBot.

anyway I pasted the key into my GitHub account's authorized keys, cloned petlogs (into the default location because fuck it), and here we are
