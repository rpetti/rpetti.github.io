---
layout: post
title: Partitioning DASD in Yast Hangs
---
I ran into this obsurd issue today. Apparently Suse 11 zLinux doesn't handle partition creation properly on DASD disks. When it gets to actually running the `fdasd` command (zLinux version of `fdisk`, as near as I can tell) it simply hangs there forever.

The work around is pretty basic, just exit out of yast, and launch `fdasd` manually.

	fdasd /dev/dasdc

The usage is basically identical to fdisk, except partitions are measured in tracks instead of blocks or sectors, and there are only two partition types instead of the multitude offered by regular partition tables.

After you've done that and written out your changes, you can jump back into yast to finish whatever you needed to do with it (formatting, creating volume groups etc.)

