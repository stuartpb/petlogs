# petlogs

## What are "pet" systems?

A "pet" system is any live system that you maintain directly, instead of instantiating as needed via reproducible builds (ie containers or NixOS). It's a nostalgic approach that most sysadmins still haven't realized is a pointless waste of time.

## What are petlogs?

Petlogs are journals of every change that's made to the running system, and *why* that change was made.

## Why keep petlogs?

Sure, you *could* just `less ~/.bash_history`, but that doesn't tell you:

- how a command was arrived upon or what documentation recommended it
- why the command was run / what it was trying to do
- what happened when the command was run
- things that happened on other accounts / shells (or GUI configuration)
- what was typed in a nano/vim/visudo session

Moreover, *no* forensically-derived analysis of a system's history is going to be of any use if the system has wedged itself into a state where such forensics would be inaccessible, ie. if you ran a command that corrupted your superblock.

For this reason, petlogs are always kept on a reliable system *outside of* the one being journaled, ie. in the cloud.

## Why keep a single repository?

I originally maintained each of these logs in separate Gists; however, having them all in one repository makes it easier to cross-reference approaches that were used in other systems when taking the same actions on another one, and also makes it easier to keep all the logs, for reference, in the same sort of layout (ie. adding frontmatter to all the logs at once in one commit).

## Tags in use

### cloud

System running as a cloud virtual machine instance, eg. a DigitalOcean droplet.

### portable

System running on some kind of removable media (usually a MicroSD card in a USB reader) for use on multiple machines.

### desktop

System running against a single, fixed set of hardware, usually off of a physically-mounter hard drive.

### chromebook

System running on a reflashed Chromebook.

### archlinux

System running Arch Linux.

### x64

System for x86_64 / AMD64 architecture.

### arm

System for ARM architecture.
