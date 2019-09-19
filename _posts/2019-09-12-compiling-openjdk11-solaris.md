---
title: "Building OpenJDK 11.0.4 on Solaris 11.4 Sparc"
layout: post
---

This is a follow up to my previous post on compiling OpenJDK on Solaris. Things have changed in the OpenJDK build requirements, so the process is slightly different.

With Oracle locking down access to their JDK, it's time to start migrating to OpenJDK. One of the major operating systems we support at my company is Solaris, and with OpenJDK not having binary distributions for that platform, it's up to us to compile it ourselves. I'm going to document the process we used here.

We'll be using Oracle Solaris 11.4 on SPARC hardware for this, though I imagine the process would be the same for x86_64 as well. I'm using OpenJDK's [official docs](https://hg.openjdk.java.net/jdk/jdk11/raw-file/tip/doc/building.html) for reference.

## Installing Requirements

First up, we'll need some tools from [opencsw](https://www.opencsw.org/)

```
pkgadd -d http://get.opencsw.org/now
/opt/csw/bin/pkgutil -U
/opt/csw/bin/pkgutil -y -i automake autoconf gmake mercurial
```

Next we need to install the compiler. Unfortunately Oracle has decided to not support GCC on Solaris, so the next steps may require a subscription with them (but don't quote me on that.) We have to install solarisstudio from Solaris' package manager.

If you don't already have solarisstudio installed, then it's likely that you will first need to add the repository to your system. Head over to [Oracle's pkg registration page](https://pkg-register.oracle.com/register/repos/), and request access to Oracle Studio. It will give you a key and a certificate. Download both and transfer to your build system.

Once they have been downloaded, add the repo to the system:

```
pkg set-publisher -k ~/pkg.oracle.com.key.pem -c ~/pkg.oracle.com.certificate.pem -G "*" -g https://pkg.oracle.com/solarisstudio/release solarisstudio
```

After that's done, you should be able to install solaris studio and confirm the version:

```
pkg install developerstudio-126
/opt/developerstudio12.6/bin/CC -V
CC: Studio 12.6 Sun C++ 5.15 SunOS_sparc Patch 152715-04 2019/08/06
```

It should be the 2018 version or later. Anything earlier than that may not work (see [here](https://bugs.openjdk.java.net/browse/JDK-8164651)). Installing the latest working version may require a support license with Oracle.

Finally install the rest of the packages needed for compilation using `pkg install`:

```
developer/gnu-binutils
x11/library/libxrandr
```

## Download JDK

Paradoxically, compiling a JDK requires a JDK. You'll need to procure JDK 10 or 11 in order to compile OpenJDK 11. For our needs, I downloaded OracleJDK 10 and extracted it to `$HOME/jdk-10.0.2`. Once we have a working OpenJDK, we can use that to bootstrap itself going forward.

## Fetch source code

If you haven't already done so, you'll need to download the OpenJDK source. This can take an extremely long time, so if you can find another trusted download source of just the latest code, I recommend doing so. If not, just pull directly from mercurial.

If you need the latest code, then use the following. For some reason the java devs duplicated their entire repo for updates, rather than just using branches or tags. Thankfully, this one actually has release tags if you only want to build a specific update.

```
hg clone https://hg.openjdk.java.net/jdk-updates/jdk11u jdk11
```

## Apply Patches

I found that some patching to the jdk11 code was necessary in order to get it to compile. This might not be needed depending on your system. It seems to be caused by a deprecated and unused symbol being removed from system headers, so it should be safe regardless.

- Patch to fix [JDK-8182035](https://bugs.openjdk.java.net/browse/JDK-8182035): [jdk11-sol114.patch](/files/jdk11-sol114.patch)

```
cd jdk11
patch -p1 < ~/Downloads/jdk11-sol114.patch
```

I would submit this patch to the ticket in question, but Java development is extremely gated so there's no way for me to access their ticketing system.

## Configure

Next, we configure the project by running the config script from the root of the source code -- as one does for any open source project -- making sure the tools from csw are in the PATH:

```
cd jdk11
export PATH=$PATH:/usr/gnu/bin:/opt/csw/bin
bash configure --with-devkit=/opt/developerstudio12.6 --with-boot-jdk=$HOME/jdk-10.0.2
```

With luck, there should be no errors.

## Building

The moment of truth! Run gmake:

```
gmake bootcycle-images
```

This will compile the JDK, then use that compiled JDK to compile the final JDK. This takes longer, but is useful to verify that everything is working. The results will be dropped into the `build/solaris-sparcv9-normal-server-release/images/jdk` directory.

## Summary

Compiling openjdk on solaris continues to be a difficult prospect, especially since it now seems that you need a support contact with Oracle in order to get the necessary compiler fixes.
