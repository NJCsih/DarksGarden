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

> Note: Swap and [[linux/nix/impermanence.md|impermanence]] are somewhat planned for here, but will require further setup
> if you want to match my config.

# Steps

## Step 1: Drive Formatting

### Formatting with gparted. Partition as: 
- >= 512 mb fat32, **flag as boot**. If you have the space I recommend 750mb to 1gb. Especially if you're going to be
    trying out different kernels and different GPU drivers. (This will be where initramfs will be).
- Whatever size lvm2 (I don't think it *needs* to be this, since LUKS will overwrite it).

### Formatting with fdiskSteps for non-gui fdisk formatting:
- `sudo fdisk /dev/\[DRIVE\]`
- `g` (create new empty gpt partition table)
- Create boot partition:
	- `n` (create new partition)
	- Sectors are often 512B, check using `TODO, what command is this? It's within fdisk, shows disk info like sector size`
	- End-Sector given as \[Start Sector\] + ( \[Desired Size\] / \[Sector-Size\] )
- Flag new partition as bootable
	- `x` for expert mode
	- `A` to toggle legacy bootable flag
	- `r` to return to normal mode
- Create main partition
	- `n` (create new partition)
	- Just use defaults to fill rest of drive, otherwise see above to calculate size. Repeat as nessecary for multiple

## Step 2: Drive Setup
This is setup to keep many important folders within their own btrfs subvolumes.

```bash
# Setup LUKS with a password
cryptsetup luksFormat <LVM2_THING_PARTITIONED_ABOVE>
# Open it up to `/dev/mapper/cryptroot`
cryptsetup luksOpen <LVM2_THING_PARTITIONED_ABOVE> cryptroot

# format the partitions
mkfs.btrfs /dev/mapper/cryptroot
mkfs.fat -F 32 /dev/<BOOT_PARTITION> # Skip if already done via gparted

# Create the subvolumes
# If you feel like making a swap subvol, that can be done here or later
mount /dev/mapper/cryptroot /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@nix
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@persist # Persistance is optional, and could be added later if desired
umount `/mnt` # Will be remounted shortly with better options

#When mounting, make sure to `mkdir` it first.
# If you're using a DE you probably need to use sudo everything.
mount -o compress=zstd,noatime,subvol=@ /dev/mapper/cryptroot /mnt

mkdir /mnt/nix
mount -o compress=zstd,noatime,subvol=@nix /dev/mapper/cryptroot /mnt/nix

mkdir /mnt/home
mount -o compress=zstd,noatime,subvol=@home /dev/mapper/cryptroot /mnt/home

mkdir /mnt/persist # Persistance is optional, and could be added later if desired
mount -o compress=zstd,noatime,subvol=@persist /dev/mapper/cryptroot /mnt/persist

mkdir /mnt/boot
mount <BOOT_PARTITION> /mnt/boot
```

## Step 4: NixOS Install

Ignore the "Do Not Edit" Warnings, Except for State Version

```bash
# Generate configuration. This will *not* keep mount parameters,
# so make sure to edit it with the ones you want
nixos-generate-config --root /mnt
```

Edit `configuration.nix` and `hardwareconfiguration.nix`. I had a duplicate boot, but
deleting it fixed it. If you're using a pre-built flake here, things will be a
lot different.

Hardware config example:
```nix
{ config, lib, pkgs, modulesPath, ... }:

{
  imports = [ ];

  boot.initrd.availableKernelModules = [
    # Yours will vary
  ];
  boot.initrd.kernelModules = [ ];
  boot.kernelModules = [ ];
  boot.extraModulePackages = [ ];


  fileSystems."/boot" = {
    device = "/dev/disk/by-uuid/XXXX..."; # UUID populated by gen-config cmd
    fsType = "vfat";
    options = [
      "fmask=0022" # Permissions stuff
      "dmask=0022"
    ];
  };

  boot.initrd.luks.devices."cryptroot" = {
    device = "/dev/disk/by-uuid/XXXX..."; # UUID populated by gen-config cmd
    allowDiscards = true;
    bypassWorkqueues = true;
  };

  fileSystems."/" = {
    device = "/dev/disk/by-uuid/d05ebcdc-704d-4c22-8666-df6e17e2276c";
    device = "/dev/disk/by-uuid/XXXX-XXXX"; # UUID populated by gen-config cmd
    fsType = "btrfs";
    options = [
      "subvol=@"
      "compress=zstd" # This not by default
      "noatime" # This also not by default, highly reccomended
    ];
  };

  fileSystems."/home" = {
    device = "/dev/disk/by-uuid/XXXX-XXXX"; # UUID populated by gen-config cmd
    device = "/dev/disk/by-uuid/d05ebcdc-704d-4c22-8666-df6e17e2276c";
    fsType = "btrfs";
    options = [
      "subvol=@nix"
      "compress=zstd" # This not by default
      "noatime" # This also not by default, highly reccomended
    ];
  };

  # @nix, @persist, @swap etc omitted, but follow suite to @home
  # If you're doing swap, enable swap here:
  # Remember fallocating swap involves some extra steps, especially on btrfs
  swapDevices = [ { device = "/swap/swapfile"; } ];

  # Enables DHCP on each ethernet and wireless interface. In case of scripted networking
  # (the default) this is the recommended approach. When using systemd-networkd it's
  # still possible to use this option, but it's recommended to use it in conjunction
  # with explicit per-interface declarations with `networking.interfaces.<interface>.useDHCP`.
  networking.useDHCP = lib.mkDefault true;
  # networking.interfaces.enp0s3.useDHCP = lib.mkDefault true;

  # Nvidia drivers can be defined here

  nixpkgs.hostPlatform = lib.mkDefault "x86_64-linux";
  hardware.cpu.intel.updateMicrocode = lib.mkDefault config.hardware.enableRedistributableFirmware;
}
```

Most of configuration.nix should be fairly self explanatory. But you should change that before building.

```bash
nixos-install
```


## Step 5: Closing Notes

I reccomend creating a config flake somewhere, likely in /home/usr, and
defining your config there rather than in /nix. Just makes it nicer.
