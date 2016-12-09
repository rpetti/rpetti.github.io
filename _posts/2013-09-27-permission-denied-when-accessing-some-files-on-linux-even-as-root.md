---
layout: post
title: '"Permission Denied" when accessing some files on linux, even as root'
---
This was a pain. Our backup software was unable to read certain Perforce versioned files from the P4ROOT (mostly .gz files). Obviously this is a fairly big issue, as it prevented us from creating any useful backups.

I checked the access control lists, ensured SELinux was disabled, and even gave the files in question 777, and still I could not read from them. We even tried running fsck multiple times with the most rigorous checking we could enable. Nothing seemed to work.

Luckily, I had `top` open in a terminal while I was mucking around and trying to read from the files manually using `dd`. The system was idle until I tried accessing one of the files, at which point a process called "`nails`" appeared for a second or two. Curious, I looked into it a bit further. The damned thing was a McAfee virus scanner! After disabling it, everything started working perfectly. The fact that it was only doing this on .gz files should have been a dead giveaway that a scanner was at fault, but then again I'd never encountered a linux system with a virus scanner installed.

Lesson learned.

