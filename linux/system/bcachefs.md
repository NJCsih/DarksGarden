---
id: bcachefs
aliases: []
tags:
  - filesystem
---

# bcachefs vs btrfs

The main feature that is holding me back from bcachefs is not being able to mount subvolumes by themselves. 
This feature is ["planned"](https://www.reddit.com/r/bcachefs/comments/1aheyn5/comment/kory9qx/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)

>  If enough people ask I'll do it eventually. 
>  But feature requests are going to take awhile to get to, still lots to do :) 

You can do bind mounts, but will leave some more clutter. btrfs does use bind mounts [interally tho](https://man7.org/linux/man-pages/man8/btrfs-subvolume.8.html).

> This is similar to a bind mount, and in fact the subvolume mount does exactly that.

TODO: write the rest of this
