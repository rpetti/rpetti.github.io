---
layout: post
title: Dual booting Ubuntu and ChromiumOS
---
Yes, I know there is a handy and rather well thought out guide [here](http://chromeos.hexxeh.net/wiki/doku.php?id=multiboot), but I thought I would post my experiences, especially given that blindly following the aforementioned tutorial resulted in my existing Ubuntu installation being wiped...
First, go read the tutorial so you know what I'm talking about. Essentially you will need to make room on your hard disk for the C-STATE and C-ROOT ChromiumOS partitions, create them, and copy them from your USB stick using dd.

I know what you are thinking, "Oh, you probably just gave dd the incorrect parameters and wiped your existing system partition." Not so, actually. I successfully copied all the data to the right partitions on my disk, and was able to boot into ChromiumOS.

The madness started once ChromiumOS started booting. I was using the latest vanilla version, which worked in live mode fine. When I booted off my main disk, however, it started performing some sort of automated recovery. That's when my Ubuntu partition got wiped.

I had 3 partitions at the time:

* `/dev/sda1` - Ubuntu
* `/dev/sda2` - C-STATE
* `/dev/sda3` - C-ROOT

Now apparently, the vanilla version of ChromiumOS doesn't like this setup much. It will always try to use the first available partition as the C-STATE. In this case it was my Ubuntu partition. Since it was the wrong label and filesystem type, ChromiumOS decided to go ahead and reformat it without confirmation.

I also tried having C-ROOT installed on sda1, and it didn't seem to like that much either. In the end, I had to set up my partitions like this:

* `/dev/sda1` - C-STATE
* `/dev/sda2` - C-ROOT
* `/dev/sda3` - Ubuntu

That seemed to work fine, with one minor exception. There was no initrd.img available on the root for some reason, and specifying the root disk using the label wasn't working either. The following grub entry was what I ended up using:

	menuentry "ChromiumOS" {
		insmod ext2
		set root=(hd0,2)
		linux   /boot/vmlinuz root=/dev/sda2 rw noresume noswap i915.modeset=1 loglevel=1 quiet
	}

Note that I had to change the root parameter, as well as completely remove the initrd line. Apparently these two problems are related (a certain something needs to be included in the initrd in order to resolve filesystem labels, and since there is no initrd it doesn't work).

So that's it! Word to the wise, it's always a good idea to do a full backup before trying any of this.

