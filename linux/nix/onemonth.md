---
id: onemonth
aliases: []
tags:
  - nix
---

# Nix, 1 month later

It's been an entire month since I picked up NixOS and gave it a shot. Here are my thoughts so far.

## Starting Out

This was definitely the roughest patch. I luckily had a decent amount of free time when I started to work on my system,
nevertheless, the learning curve was quite steep.

The issues this imposed on my system could have been mitigated if I had spent more time messing with Nix(OS) in a
VM or with home manager. Luckily, it didn't take too long to start to feel comfortable in the Nix language.

The biggest help was definitely seeing other people's configurations, and perusing the [discourse](https://discourse.nixos.org/).

## When things broke

I only had 1 major "ah \*\*\*\*" with NixOS. The other one was fairly minor.

### The boot catastrophe

A week in I decided I would like to move to the grub boot loader. I wanted this for 1 reason: submenus. I wanted to mess with 
[[linux/nix/specialisations.md|specialisations]], and didn't want to have to page through a hundred systemd menu entries.

> Notice: This may have been fixed by the time you're reading this. Follow [this PR](https://github.com/systemd/systemd/pull/28084)
  that I stumbled upon. Literally submenus were the only reason I went to grub.

So: I change my boot loader, I rebuild config. Everything is happy. I go to restart my computer, wait a minute, and all of the 
sudden I'm in Windows. Weird, but not that unexpected. I turn it off, go to pull up the boot menu, it blinks, and then I'm back in Windows.

These are all the things I tried, but I was completely locked into Windows:
- Booting from a live install
- Trying to set a USB as the default boot 
- Resetting bios
- Removing the boot flag from the boot partition
- Going to bed

Finally, I decided to just wipe the boot partition (mind you, this was all in Windows!!! WHICH WAS AWFUL TO MESS WITH PARTITIONS), and then
my system went back to normal. All I had to do now was reinstall the boot loader, and then it worked.

So, in short, either completely remove your boot partition when switching boot managers, or at least wipe it. This was probably
an issue with MSI, but still quite annoying.

### The SOPS password issue

When I was preparing to set up my system for impermanence, I went to move my password to a hashed SOPS file. All went well, until I wanted
to move the configuration, and I made the poor decision of not having it on the persist partition. A quick boot to the recovery drive, and
all was well.

## The rest of the time

I very quickly understood why so many people love NixOS. As I continued down my journey I saw a bunch of positives from Nix, here is a short list:

- You build it once, it works forever (:tm:).
- Increased resilience
- `nix shell nixpkgs#<package>` is THE BEST. Try out a package, and just ignore it for the garbage collector to clean it up if you don't like it
- Reproducible dev environments
- Systemd services are incredibly easy to set up
- Shell scripts can be added to PATH very easily.
- Shebangs have super powers with `nix shell`.
- Custom packages are very satisfying to set up, and allow you to update fairly easy.
- Dependencies are not shared (this is also a negative), you can install whatever versions of packages (most of the time).
- SOPS. Super cool.
- Multiple systems in one config.
- Custom live media.
- So much configuration can be done in your Nix config.

The biggest was me having less fear about breaking stuff. This caused me to experiment with the Linux Zen kernel, impermanence,
compositors, breaking versions of software, and more. You don't have to do this if you use Nix, but it is a very nice bonus.

Here are a few negative things I came across: 

- It does take longer to learn Nix, and to build a system you're happy with.
- The community is growing, and so there are some growing pains.
- Flakes should not be used for super complicated stuff (it's experimental), but they work very well for system configurations.
- Documentation is still coming along. An official wiki is in the works.
- Sometimes it's annoying to have to do a full rebuild to test one thing.
- Dependencies are not shared. This becomes an issue with the linker, and that's a full rabbit hole. 
  It is quite a pain, but there are ways to circumvent the issues, and you get better over time.

# Other Good Materials

Blog posts:

- [This Cute World](https://thiscute.world/en/posts/my-experience-of-nixos/)

Configurations:
- [Jake Hamilton](https://github.com/jakehamilton/config)
- [khaneliman](https://github.com/khaneliman/khanelinix)
- [EmergentMind](https://github.com/EmergentMind/nix-config)
