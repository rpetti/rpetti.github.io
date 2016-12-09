---
layout: post
title: 100% cpu usage in Oneiric (Ubuntu 11.10)
---
I recently upgraded my work PC from Natty to Oneiric, and discovered that whenever I had a window open that __wasn't__ maximized, it's backend process and xorg would use up 100% of the cpu.

I was nearly about to rebuild my machine when I discovered that it only happened for non-maximized __GTK__ windows. As a last ditch effort, I ran `gtk-theme-switch2`, and switch my theme. Magically, this fixed the problem!

Part of the issue was that I'm running in Fluxbox, not Gnome or Unity. I think somehow it got the theme messed up during the upgrade, and since neither Gnome nor Unity had the chance to set it to a correct theme, it borked.

