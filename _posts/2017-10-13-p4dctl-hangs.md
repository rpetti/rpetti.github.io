---
title: "p4dctl hangs on startup"
layout: post
---

Perforce has introduced a rather neat utility called [p4dctl](https://www.perforce.com/perforce/r17.1/manuals/p4dist/Content/P4Dist/appendix.p4dctl.html) that assists with managing the various services (p4d, p4p, p4broker) that you might want to run on a linux machine. They even provide deb and rpm packages for it. Unfortunately, their RPM contains an init.d script that is incompatible with systemd, which is what comes with RHEL/CentOS >= 7. This may also be an issue in modern Debian based systems, but I haven't tested it.

Here's an example of what I'm talking about:

	[root@bg-perforce--proxy-d001 ~]# /etc/init.d/perforce-p4dctl start
	Starting perforce-p4dctl (via systemctl):  
	#Just hangs here forever

The issue is that their init.d script has a pidfile option in the LSB header that doesn't point at a real file. The SysV compatilility functions of systemd uses this header to generate a wrapper systemd service that can be controlled using systemctl.

	#!/bin/bash
	#
	#	/etc/rc.d/init.d/perforce-p4dctl
	#
	#	Starts/stops Perforce related services
	#
	# chkconfig: 2345 80 20
	# description: perforce-p4dctl supports multiple Perforce services on a single \
	#              host and allows those services to be owned by different \
	#              users while still providing root with overall control. \
	#              see 'man p4dctl' for more information
	# config: /etc/p4d.conf
	# pidfile: /var/run/... (one for each managed service)
	
	# Source function library.
	. /etc/init.d/functions
	
	...
	...

As a result, systemctl waits for that pid file to be created indefinitely, as shown by journalctl:

	Oct 13 21:12:52 bg-perforce--proxy-d001 systemd[1]: PID file /var/run/... (one for each managed service) not readable (yet?) after start.

Simply removing the `# pidfile:` line entirely and running `systemctl daemon-reload` allows the service to be started/stopped by systemd without any issues. I've already notified Perforce support of this problem, so hopefully I can convince them it's an actual problem that needs to be fixed, otherwise there will probably be a lot of confused admins out there trying to use this on a modern RHEL/CentOS system for the first time.

*Update* 2018-05-16: Perforce still has not resolved the issue. I've pinged them again, and thankfully they were able to partially reproduce the issue and will hopefully fix it in the next release (2018.3?)

*Update* 2018-11-26: I recently received a notification from Perforce that this has been fixed, so hopefully it's resolved in the next release.