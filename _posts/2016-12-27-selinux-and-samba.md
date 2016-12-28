---
layout: post
title: SELinux and Samba Woes
---

I understand why SELinux exists, but I still hate working with it sometimes... You'd think that after years of running into bizarre issues with nonsensical error messages I'd learn to check SELinux first, but apparently I'm a slow learner.

This latest one comes while trying to access files that I've shared over Samba. Apparently, SELinux contexts are not inherited from parent directories in certain situations. I did the necessary setup of the initial folder, but I couldn't access any of the files in it.

I had to run this on the shared folder to recursively allow Samba access to the files in the share:

	chcon -R -t samba_share_t /data

Hopefully this won't need to be executed every time I add files to it.
