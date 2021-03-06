---
layout: post
title: Value too large for defined data type
---

I recently ran into an issue (on Christmas Day, in fact) with certain binaries being unable to access files over NFS.

Here's the configuration I'm running:

* RedHat Enterprise Linux 5 32-bit (NFS Client)
* RedHat Enterprise Linux 7 64-bit (NFS Server)

When some processes tried to access files over NFS from the RHEL 5 client, it would fail with `Value too large for defined data type`. The solution was to recompile every failing executable with `make CPPFLAGS=-D_FILE_OFFSET_BITS=64`. This enables large file support, which would obviously allow access to files over 2GB, but weirdly also allows access to files hosted over NFSv3 from 64-bit systems.

I really wish I didn't need to work with such antiquated systems all the time, but it is what it is.

_Update:_ It would seem I'm not entirely correct here. It's not actually the NFS version that matters, but how large the inode numbers are on the shared filesystem. XFS for instance, uses 64-bit inode numbers when over a certain volume size, which causes the above error. Another solution that I eventually implemented was reformatting the shared volume using EXT4 instead.
