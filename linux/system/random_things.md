---
id: random_things
aliases: []
tags: []
title: Random things I don't want to forget
---

# Expanding btrfs partition within LUKS

You can use KDE Partition manager or gparted to expand the LUKS partition. This will not
expand the drive within it. After expanding the encrypted partition, mount your btrfs drive
(in this case it will be on `/mnt/btrfs`) and then run:

```
btrfs filesystem resize max /mnt/btrfs
```

This will resize the drive to take up all possible space, pretty sweet.
