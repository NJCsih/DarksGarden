---
id: installation
aliases: []
tags: []
title: Installation
---

This is to document my installation process of [[linux/nix/index.md|NixOS]]. I think that installing
without the graphical installation is good practice. Installing without a graphical tool also makes
it easier to repair your install in case something breaks.

Doing it without a graphical install also gives you more control over partitions and lets you ensure
that your system is set up exactly as you want it.

This guide is specifically *my* install. I have LUKS encryption, so if you want to adapt this, it will
probably look different.

> Note: This does not handle [[linux/nix/impermanence.md|impermanence]]. You'll need to add another subvolume
> if you want to match my setup.

# Steps

## Step 1: Partition the Disks

When mounting, make sure to `mkdir` it first. If you're using a DE you probably need to use sudo for everything.

Go to gparted. Partitions are: 
- >= 512 mb fat32, **flag as boot**. If you have the space I recommend 750mb to 1gb. Especially if you're going to be
    trying out different kernels and different GPU drivers. (This will be where initramfs will be).
- Whatever size lvm2 (I don't think it *needs* to be this, since LUKS will overwrite it).

## Step 2: Generate Config and Installation 

```bash
# Setup LUKS with a password
cryptsetup luksFormat <LVM2_THING_PARTITIONED_ABOVE>
# Open it up to `/dev/mapper/cryptroot`
cryptsetup luksOpen <LVM2_THING_PARTITIONED_ABOVE> cryptroot
mount /dev/mapper/cryptroot /mnt

# Create the subvolumes
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@nix
btrfs subvolume create /mnt/@home
# umount `/mnt`

mount -o compress=zstd,noatime,subvol=@ /dev/mapper/cryptroot /mnt 

mkdir /mnt/nix
mount -o compress=zstd,noatime,subvol=@nix /dev/mapper/cryptroot /mnt 

mkdir /mnt/home
mount -o compress=zstd,noatime,subvol=@home /dev/mapper/cryptroot /mnt 

mkdir /mnt/boot
mount <BOOT_PARTITION> /mnt/boot

# Generate configuration. This will *not* keep mount parameters,
# so make sure to edit it with the ones you want
nixos-generate-config --root /mnt
# Edit configuration.nix/hardware configuration. I had a duplicate boot,
# but deleting it fixed it. If you're using a pre-built flake here,
# things will be a lot different.
vim /mnt/etc/configuration.nix

nixos-install
```
