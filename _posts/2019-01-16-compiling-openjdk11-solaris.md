---
title: "Building OpenJDK 11 on Solaris 11 Sparc"
layout: post
---

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

After that's done, you should be able to install solaris studio. 12.4 was the only version that worked for me, and I also needed to unlock the python-26 version in order for ss12.4 to even be installable on my system:

```
pfexec pkg change-facet facet.version-lock.runtime/python-26=false
pfexec pkg change-facet facet.version-lock.library/python-2/tkinter-26=false
pkg install solarisstudio-124
```

## Download JDK

Paradoxically, compiling a JDK requires a JDK. You'll need to procure JDK 10 or 11 in order to compile OpenJDK 11. For our needs, I downloaded OracleJDK 10 and extracted it to `$HOME/jdk-10.0.2`. Once we have a working OpenJDK, we can use that to bootstrap itself going forward.

## Fetch source code

If you haven't already done so, you'll need to download the OpenJDK source. This can take an extremely long time, so if you can find another trusted download source of just the latest code, I recommend doing so. If not, just pull directly from mercurial.

If you need the latest code, then use the following. For some reason the java devs duplicated their entire repo for updates, rather than just using branches or tags. Thankfully, this one actually has release tags if you only want to build a specific update.

```
hg clone https://hg.openjdk.java.net/jdk-updates/jdk11u jdk11
```

If you only need the 11.0.0 GA code, then use the following repo:

```
hg clone https://hg.openjdk.java.net/jdk/jdk11 jdk11
```

## Apply Patches

I found that some patching to the jdk11 code was necessary in order to get it to compile. This might not be needed depending on your system. It seems to be caused by a deprecated and unused symbol being removed from system headers, so it should be safe regardless.

- Patch to fix [JDK-8182035](https://bugs.openjdk.java.net/browse/JDK-8182035): [jdk11-sol114.patch](/files/jdk11-sol114.patch)

```
cd jdk11
patch -p1 < ~/Downloads/jdk11-sol114.patch
```

I would submit this patch to the ticket in question, but Java development is extremely gated so there's no way for me to access their ticketing system.

## Creating a Sysroot

It seems that OpenJDK 11 does not actually compile against Solaris 11.4 headers. As a result, we need to create a sysroot containing the required headers and libraries:

```
cd $HOME
pkg image-create sysroot
pkg -R sysroot set-publisher -g http://pkg.oracle.com/solaris/release/ solaris
pkg -R sysroot install --accept entire@0.5.11-0.175.1 release/name@0.5.11-0.175.1 consolidation/osnet/osnet-incorporation@0.5.11-0.175.1
pkg -R sysroot install --accept system/core-os@0.5.11-0.175.1
pkg -R sysroot install --accept system/header@0.5.11-0.175.1
pkg -R sysroot install --accept developer/base-developer-utilities@0.5.11-0.175.1 developer/gnu-binutils@2.21.1-0.175.1 developer/macro/cpp@0.5.11-0.175.1 system/linker@0.5.11-0.175.1 developer/base-developer-utilities@0.5.11-0.175.1 developer/gnu-binutils@2.21.1-0.175.1
pkg -R sysroot install --accept print/cups@1.4.5-0.175.1 system/library/freetype-2@2.4.9-0.175.1 library/zlib@1.2.3-0.175.1 x11/header@0.5.11 x11/library/libxtst@1.2.1-0.175.1 x11/library/libxi@1.6.1-0.175.1 system/library/fontconfig@2.8.0-0.175.1 system/picl@0.5.11-0.175.1
```

OpenJDK 11 has scripts for these in `make/devkit` in it's source tree, but they don't work at all due to _completely_ incorrect version identifiers...

Finally, since OpenJDK 11's `--with-sysroot` configure option does not totally work correctly, we need to do an additional hack in order to force the compilation to use the system headers from sysroot. I haven't found any other way around doing this, so if you have an idea please let me know. I already tried chroot (which just broke bash) and -I compilation flags (which wouldn't preclude the `/usr/include` path).

```
mv /usr/include /usr/include.bak
ln -s $PWD/sysroot/usr/include /usr/include
```

Just don't forget to undo this when you are done!

## Configure

Next, we configure the project by running the config script from the root of the source code -- as one does for any open source project -- making sure the tools from csw are in the PATH:

```
cd jdk11
export PATH=$PATH:/opt/csw/bin
bash configure --with-devkit=/opt/solarisstudio12.4 --with-boot-jdk=$HOME/jdk-10.0.2 --with-sysroot=$HOME/sysroot
```

With luck, there should be no errors.

## Building

The moment of truth! Run gmake:

```
gmake bootcycle-images
```

This will compile the JDK, then use that compiled JDK to compile the final JDK. This takes longer, but is useful to verify that everything is working. The results will be dropped into the `build/solaris-sparcv9-normal-server-release/images/jdk` directory.

## Revert header hack

Make sure to restore the original system headers!

```
rm /usr/include
mv /usr/include.bak /usr/include
```

## Conclusion

This whole process was a lot harder than it really should have been. Considering Oracle develops OpenJDK, you would think that they would have more robust support for compiling on their own platform. The cynic in me wants to believe that this was an intentional marketing decision to force Solaris customers into paying for OracleJDK, but the realist in me knows that this was likely a result of incompetence, laziness, or just a simple lack of time. Regardless, I really hope they open up OpenJDK development and bug reporting (it's extremely gated right now) so issues like this can be identified and resolved by the community at large.