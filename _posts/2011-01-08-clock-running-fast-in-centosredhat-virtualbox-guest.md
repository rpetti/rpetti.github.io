---
layout: post
title: Clock running fast in CentOS/RedHat VirtualBox Guest
---
Just ran into this issue while setting up a CentOS 4.7 test virtual machine for some debugging work. After some digging around on the webernets, I found this little [gem.](http://www.virtualbox.org/manual/ch12.html#id497868)

Basically, the kernel is hardcoded for a higher timer frequency than VirtualBox is providing. 10 times in fact. The simple solution is to add `divider=10` to the kernel options. This slowed the clock of the guest down to a more accurate level.

