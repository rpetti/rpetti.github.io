---
title: "NFS accesses randomly causing 'Read-only filesystem' errors"
layout: post
---

I'm currently setting up some new systems in a remote datacenter, and ran into random write failures on an NFS server I just set up. Surprisingly, the issue appeared to be with DNS resolution of the client's hostname, which makes perfect sense when you realize that we are limiting write access to specific _hostnames_.

Server's /etc/exports

    /data -sync,no_acl austsbld*(rw) hydwcmbld*(rw) *

Client's hostname: `hydwcmbldlin64`

This should have access right? Well, only if the server can actually verify that the client is the hostname that's included. Unfortunately, the IT in the remote office does not do a very good job of keeping their DNS tables clean, and I found a duplicate entry in the reverse lookup zone:

    $ nslookup 10.AA.BB.CC
    Server:         10.XX.XX.XXX
    Address:        10.XX.XX.XXX#53

    CC.BB.AA.10.in-addr.arpa        name = hydwcmbldlin64.REDACTED.
    CC.BB.AA.10.in-addr.arpa        name = hyd43-win-ha1.REDACTED.

Removing this duplicate entry from the reverse lookup table almost immediately resolved the write access problem.