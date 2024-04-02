---
id: btrfs
aliases: []
tags:
  - filesystem
date: "2024-04-01"
title: btrfs
---

[btrfs](https://btrfs.readthedocs.io/en/latest/) (b-tree filesystem) is a modern file system with a ton of useful features.
This page will not be an in depth dive into btrfs, but instead an easy to read document on some btrfs features I have found.

> NOTE: I am not a professional filesystem expert, some of this information may be wrong, but I'm trying to continue to
  add to this page as I learn more.

# Features

## Copy on Write (CoW)

Copy on write is actually super cool. Very simply it means that on write files get copied to a new location first.
This has a few merits:

### Reflinks and Fragmentation

When copying and pasting a file, usually the system copies the entire file into the new location. btrfs (usually) does not do this.
It instead creates a reference link (reflink) that points to the original location. You can think of this as a pointer. This means 
copying and pasting is essentially free.

Each file is made up of "chunks" of data, and each time you write to the file, it writes to a new place first, then changes the link.
If there are no "pointers" to the original data, it can then be overwritten, but if there still are, the file will continue to get split (fragmented).

From the btrfs [documentation](https://btrfs.readthedocs.io/en/latest/Defragmentation.html):

> The fragmentation is caused by rewrites of the same file data in-place, that has to be handled by creating a new copy that may lie on a distant location on the physical device.

You can run [defragmentation](https://btrfs.readthedocs.io/en/latest/Defragmentation.html) on btrfs which will try to keep the same file
in the same spot. This is a costly operation and *should be done with caution*. On SSD's this isn't as necessary because order does not 
affect speed. Defragmentation can lead to corruption and it **removes reflinks** (i.e. BIGGER FILES, and SNAPSHOTS WILL EXPAND).

#### Side Note: Balancing

With btrfs you can balance your file system. This is similar to defragmentation, but it just condenses the blocks of storage.

btrfs has 3 types of blocks:
- System: things btrfs has to keep track of the filesystem
- Metadata: file information (reflinks, data location, compression, atime, mtime...)
- Data: actual file data

btrfs allocates these blocks in chunks (usually 1gb for data) and will then fill them up and allocate more as needed. If you write
and delete a lot, some of these can be fairly sparse and balancing can condense them back together.

**Only balance data because there can be problems with balancing metadata**. Check out [this article](https://wiki.tnonline.net/w/Btrfs/Balance) for more information
on actually balancing.

### Snapshots

#### The Good

Snapshots are a way to make a full backup of a subvolume. They use reflinks so they initially take zero space and then
only new writes take up space. This allows for a really elegant full system backup.

#### The Bad

> From a post from [dalto](https://forum.endeavouros.com/t/my-testing-of-bcachefs/49607/) (not a professional filesystemer either, but interesting points).

The main problem with snapshots (that I can gather) is the lack of metadata stored within them. This means tools like [snapper](http://snapper.io/) are
needed to help keep track of the snapshots. The biggest problem arises when you restore a snapshot you lose track of the parent and the other
snapshots become orphaned.

## Subvolumes

Subvolumes are awesome! They are essentially folders with superpowers. My favorite part of them is that you can mount them individually.
This means my btrfs layout looks like:

```
/@ -> mounts to /
/@home -> mounts to /home
/@persist -> mounts to /persist
/@nix -> mounts to /nix
```

You can move these around all you want (literally like folders) and delete and recreate them. This gives superpowers to [[content/linux/nix/impermanence.md|NixOS impermanence]].

Each subvolume mount can have different options and can be snapshotted independently.

## Compression

Compress is also quite amazing. Counter-intuitively, if you compress smart you can actually increase read and write speeds. If you think about it,
getting data from a drive is somewhat slow (especially compared to RAM and CPU data throughput), so with compression you can transfer less data and
then expand it.

btrfs compression is transparent, meaning that from the end-user perspective the file is just a normal file. The details of compression are
completely abstracted away by the OS. So you can just leave it and forget.

### force-compress

I stumbled upon some differing opinions on if you should add the option `force-compress`. After using btrfs for a while I can say I do not recommend it.
When btrfs goes to compress a file, it first checks the first chunk of data and if it cannot be compressed well, it marks the entire file as `NO_COMPRESS`.
This is done because files that don't compress well can just hog up CPU time and sometimes take more space.

Force compress removes this check and has every file compressed. In practice, this will probably lead to very few benefits, and you can always force
compress a file by itself. Maybe enable this if you really want high compression ratios and don't care at all about speed.

### Use cases and which file compression you should choose

I'm not going to tell you which compression type you use, but here are a few things to keep in mind.

If you are going to compress your main drive you should probably choose something with high compression and decompression speeds. A high compression ratio should come
second.

If instead you want something like a media drive where you will write fairly infrequently, but want to be able to be able to still read fast, choose something
with a higher compression ratio, still with good read speeds.

With compression you quickly encounter diminishing returns if you're striving for larger compression ratios. As always, do some research. I have been rocking
the default `zstd:3` compression and have not encountered any issues.

## Raid

I have not tried any RAID configurations with btrfs. From the random things I have stumbled upon (**DO YOUR OWN RESEARCH**) it seems that btrfs is good 
for RAID0 and RAID1. RAID5/6 have really bad bugs. If you're looking to RAID, from what I can tell [[content/linux/system/bcachefs.md|bcachefs]] has good
large storage support and can do RAID's well. It has a lot of the same features, and is getting better by the day.

# Other Sources

- [Forza's Ramblings](https://wiki.tnonline.net/w/Category:Btrfs)
- [Documentation](https://btrfs.readthedocs.io/en/latest/)
