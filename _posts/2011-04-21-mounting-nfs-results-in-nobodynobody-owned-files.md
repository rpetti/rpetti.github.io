---
layout: post
title: Mounting NFS results in nobody:nobody owned files
---
So I ran into this problem earlier this week. Basically we have a Solaris 10 server hosting files over NFS. The NFS server that comes with Solaris 10 supports NFSv4, but doesn't seem to include idmapd, which is responsible for mapping user and group ids. Everything I've read suggests that idmapd is required on both the client and server in order for it to work correctly. Since I had no real desire to screw with the server configuration (and since other machines could mount it correctly) I kept searching.

The solution I found is a bit hacky, but it works for me. Basically it boils down to adding `nfsvers=3` to the mount parameters. In my case, I was using automount, so I just added the option to every line in my /etc/auto.master. This just forces it to mount using NFSv3, which works because of it's simplicity, but is possibly more insecure (and not all NFS servers will support it).

If possible, you should really just consider doing things the proper way, and getting idmapd set up on your servers and clients. This is just a quick work-around for when you are backed into a corner like I am.

