---
layout: post
title: Resizing Solaris Partitions
---
I'm currently in the process of resizing a partition in Solaris 10. So far, the instructions that I've found have been quite incorrect, so I'm documenting the steps I'm taking here. In my particular case, I'm resizing a single partition on a non-root disk after increasing it's size through VMWare.

**This is whole process stupidly dangerous, and possibly just plain stupid. Take a backup before trying it!**

Partitions in Solaris work quite a bit differently from partitions in other *nix platforms. First we have the partition. This is what we normally resize using utilities such as gparted. Within the Solaris partition, are slices. These can be thought of as 'sub partitions' that contain the filesystems to be mounted in the OS. In fact, Solaris often refers to slices as partitions, which can get a bit confusing at times. First we need to resize the overall Solaris partition, then resize the slice(s) within it.

It's worth noting that my case is exceptionally simple. I have a single filesystem, on a single slice, on a single partition, on a single disk separate from the root one. I don't recommend trying any of these steps on a root disk or root partition. Consider yourself warned.

First, we need to find out the device name:
    
    $ df -h /disk1
    /dev/dsk/c2t1d0s0      148G   112G    34G    77%    /disk1

Right now, we're interested not in the slice being mounted but the partition containing the slice. In this case it is c2t1d0p0. I'm sure there's some other way to figure this out, but in my case I only have one slice and one partition on this particular disk (c2t1d0), so it was easy to figure it out.

Unmount all slices:
    
    # umount /disk1

Dump the disk geometry:
    
    # fdisk -G /dev/rdsk/c2t1d0p0 > /tmp/geom

Then dump the current partition table:
    
    # fdisk -W /tmp/ptbl.tmp /dev/rdsk/c2t1d0p0

I recommend backing up the partition table at this point, in case you need it later.

Also, you will want to record the current slice map. Solaris has a command to help you do this, but it refers to slices as 'partitions'. These partitions are actually slices in the Solaris partition that (should) encompass the entire disk, and are not to be confused with the disk's partition table that you will modify using fdisk. You can print the current slices using the `format` command, selecting your disk, and printing the partition map ('p' from the partition menu). It should look something like this:
    
    Part      Tag    Flag     Cylinders         Size            Blocks
      0 unassigned    wm       1 - 19577      149.97GB    (19577/0/0) 314504505
      1 unassigned    wm       0                0         (0/0/0)             0
      2     backup    wu       0 - 19577      149.98GB    (19578/0/0) 314520570
      3 unassigned    wm       0                0         (0/0/0)             0
      4 unassigned    wm       0                0         (0/0/0)             0
      5 unassigned    wm       0                0         (0/0/0)             0
      6 unassigned    wm       0                0         (0/0/0)             0
      7 unassigned    wm       0                0         (0/0/0)             0
      8       boot    wu       0 -     0        7.84MB    (1/0/0)         16065
      9 unassigned    wm       0                0         (0/0/0)             0

Keep this somewhere safe for referencing later.

Edit the ptbl.tmp file you created. At the top, you'll see your disks geometry. At the bottom is the layout. The ONLY field you are interested in modifying at the bottom is the number of sectors (Numsect).
    
    * /dev/rdsk/c2t1d0p0 default fdisk table
    * Dimensions:
    *    512 bytes/sector
    *     63 sectors/track
    *    255 tracks/cylinder
    *   22192 cylinders
    * Id    Act  Bhead  Bsect  Bcyl    Ehead  Esect  Ecyl    Rsect    Numsect
      191   128  0      1      1       254    63     1023    16065    314552700

In my case, I needed to expand the partition to fill the entire disk. I did this by taking the number of cylinders, subtracting one (since the partition starts at cylinder one and not 0 (Bcyl)), multiplying that number by tracks/cylinder, and multiplying that by sectors/track. So the new Numsect value is: 22191*255*63 = 356498415.

Once that's been changed, write the new partition table to disk:
    
    # fdisk -S /tmp/geom -F /tmp/ptbl.tmp -I /dev/rdsk/c2t1d0p0

You can run fdisk normally (`fdisk /dev/rdsk/c2t1d0p0`) in order to verify your changes. Once that is done, run `format` again, since the solaris partition (slices) map may have been lost. In my case, slice 0 lost it's information, so I recreated it to start at the same start cylinder as before (1), and take up the rest of the disk. Modify it using '0' from the partition menu, and save the new map using the 'label' command from within the partition menu.
    
    Part      Tag    Flag     Cylinders         Size            Blocks
      0 unassigned    wm       1 - 22188      169.97GB    (22188/0/0) 356450220
      1 unassigned    wm       0                0         (0/0/0)             0
      2     backup    wu       0 - 22188      169.98GB    (22189/0/0) 356466285
      3 unassigned    wm       0                0         (0/0/0)             0
      4 unassigned    wm       0                0         (0/0/0)             0
      5 unassigned    wm       0                0         (0/0/0)             0
      6 unassigned    wm       0                0         (0/0/0)             0
      7 unassigned    wm       0                0         (0/0/0)             0
      8       boot    wu       0 -     0        7.84MB    (1/0/0)         16065
      9 unassigned    wm       0                0         (0/0/0)             0

The slice should now have been resized. Next is to verify that no data was lost:
    
    # fsck /dev/rdsk/c2t1d0s0

And finally, grow the filesystem to take up the rest of the slice:
    
    # growfs /dev/rdsk/c2t1d0s0

And that's it! Hopefully this helps people understand Solaris partitioning schemes a little bit better. I learned quite a bit while figuring this out.

