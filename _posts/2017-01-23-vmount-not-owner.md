---
layout: post
title: "'vmount: Not owner' when mounting Linux NFS export on AIX"
---

I recently ran into this issue while trying to mount an NFS export on a linux server on an AIX server.

	$ mount linuxserver:/data /mnt/data
	mount: giving up on:
	 linuxserver:/data
	vmount: Not owner

As usual, AIX has extremely nonsensical and useless error messages. The easiest solution for me was to add the 'insecure' option to the NFS export on the linux server:

	/data * (rw,sync,insecure)

This allowed it to mount at least, but I don't know what other effects it would have, if any.
