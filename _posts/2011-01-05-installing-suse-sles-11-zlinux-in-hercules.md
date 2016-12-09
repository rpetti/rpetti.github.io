---
layout: post
title: Installing Suse SLES 11 zLinux in Hercules
---
What follows is a chronicle of my efforts in getting Suse SLES 11 installed in Hercules. Hopefully it will be of use to others who need to deal with this ridiculous 31 bit platform.

Recently I received a request to build a product for a rather ridiculous platform, S380. Needless to say, this is one of those IBM platforms where they try to create their own standards, breaking all conventions when doing so, and selling it at 100x the cost.

Ranting aside, we have some customers using our product on this platform. The builds they are running were created on a machine that was loaned out to us, but we no longer have access to it.

Enter Hercules. This is a straight emulator for the S380/zSeries platform. Since the platform is so different from anything that's considered sane or normal, the emulator does things the hard way by emulating the entire processor and machine. Modern virtualization solutions tend to perform tricks that let the hardware of the vm host run the code directly. Unfortunately that isn't an option here, so anything running in Herc will be dog slow. I'm not trying to knock the developers, because they did a good job, that's just the reality of the situation.

Setup
=====

First off I downloaded Suse SLES 11 zLinux from the Suse [webpage](http://www.novell.com/linux/). There are two DVDs, but you'll only need the first one in order to do a basic installation.

Next, you need to install Hercules on your host system. I've used Ubuntu Server 10.04 LTS and 8.04 LTS. For the latter, you'll need to compile Hercules from source, because I ran into networking issues with Herc versions lower than 3.05. For 10.04, you can simply use the versions in the repos:

	$ sudo apt-get install hercules

You'll also need to install `apache2`. The reason being is that The SuSE installation disk does not come with mountable tape images, so you'll need to do a network install. This requires a working http server.

	$ sudo apt-get install apache2

Next we need to mount the disk image we downloaded:

	$ sudo mkdir -p /var/www/install/DVD1
	$ sudo mount -o loop SUSE-DVD1.iso /var/www/install/DVD1

Now it's time to setup Hercules. This is the `hercules.cnf` I used to set up the system.

	ARCHMODE ESAME
	OSTAILOR QUIET
	LOADPARM 0120....
	CPUSERIAL 000069
	CPUMODEL 2064
	LPARNAME HERCULES
	MAINSIZE 512
	SYSEPOCH 1900
	PANRATE FAST
	NUMCPU 2
	CNSLPORT 3270
	XPNDSIZE 0
	
	HTTPROOT /usr/local/share/hercules/
	HTTPPORT 8081 NOAUTH
	
	0120    3390    /srv/zlinux/root.0120
	0121    3390    /srv/zlinux/swap.0121
	0E20.2  3088    CTCI 10.1.1.2 10.1.1.3
	#Leave commented for now, we'll come back to it later.
	#0E26.2  3088    LCS 172.29.9.148 -m 3E:31:C5:59:36:DC

I won't go into it too much, as the options are described on the hercules webpage. Note, however that the devices `0120` and `0121` are pointing towards two files. These are going to be your disk images. Feel free to change them to whatever location you want. Next, we'll actually create the disk images:

	$ sudo dasdinit -lfs root.0120 3390 ROOT 8000
	$ sudo dasdinit -lfs swap.0121 3390 SWAP 500

The last number in the command is the size of the disk in cylinders. Each cylinder is approximately 800KB (852480 bytes/cyl to be exact) though I found that there was slightly less usable disk space than I expected. Regardless, 8000 cylinders should be enough for your installation. You can always add more disks later.

Installation
============

Now we start the actual installation! Go into the directory with your disks and your `hercules.cnf` file, and run the following.

	$ sudo hercules -f hercules.cnf

This should load the hercules ncurses gui. Unfortunately, there doesn't seem to be a headless mode, so in order to run this as a server you'll need to do some magic with `screen`. We'll get to that later. For now, start the installation:

	> ipl /var/www/install/DVD1/suse.ins

The rest of the process is pretty straight forward. Choose `Start Installation`, `Network`, `HTTP`, and `Channel to Channel`. By default, all commands typed are interpreted by hercules, not the guest operating system, so you'll need to escape everything you type by prepending it with a period. So to select option 2, you need to type `.2`

After you choose Channel to Channel, you need to select the read and write channels. The defaults work, so just use those:

	.0.0.0e20
	.0.0.0e21

Select `Compatibility Mode` and No for automatic configuration via DHCP. We need to set up the ip addresses manually. If you used my config file, specify `10.1.1.2` and `10.1.1.3` for the local and remote side respectively. Next it will ask you for your nameserver, just give it the IP address of whatever your host is configured to use.

Now you need to enter the IP address of the webserver. Give it `10.1.1.3` (see where we're going here?) Then give it the installation media path of `install/DVD1/`. Choose SSH as the display type, and give it a simple temporary password. It should now load the installation. Once it's loaded, some instructions will appear on the screen. From another terminal on the host machine, run:

	$ ssh root@10.1.1.2
	# yast

Proceed through the installation as you normally would for any other system. When you get to the disk activation screen, select DASD and activate both disks that you've set up. There are options for both activating and selecting, make sure you select __and__ activate both disks. It'll ask you to format them as well, so do so.

Once you get to the installation summary, it will likely tell you that the automatic partitioning couldn't find a partitioning scheme on it's own. Enter the partitioner and set up the disks how you like. I used the root disk as a single ext3 partition mounted on `/`, and the swap disk as a single swap partition. Once you are done, you can start the actual installation process. This took about 5-6 hours to complete for me, so leave it for the night and come back to it in the morning. At this point it will automatically reboot and ask you to ssh back into the machine and finish up the installation by running a command. Follow the onscreen instructions, and the install will be complete!

To boot into the new environment (if it didn't do so automatically) type the following into the hercules prompt after starting it up:

{% highlight plain %}
ipl 0120
{% endhighlight %}

You can also add this to a `hercules.rc` file in the working directory, which hercules will read whenever it's started up. Perfect for automatically starting it up in `screen` on bootup. ;)

Post-Install Configuration
==========================

Networking Hacks
----------------

You've now got a working zLinux installation with no networking capabilities. If this is all you need, then you are done. If you need networking, then we'll need to set up a bridge.

First, lets get the guest operating system set up. Shut down Hercules, and edit the `hercules.cnf` file. Uncomment the 0E26.2 interface line, and replace the IP address with the IP you want the guest operating system to use on your network. This should not be the same as the host system! It should be totally separate, but still in your netmask.

Start up the server as describe above. SSH to the zLinux instance and run yast as you've done previously. From here, you can set up your network device as you normally would. Choose the `0.0.0e26 device (IBM OSA2 Adapter)` and edit it. Set it up exactly as you would a computer plugged directly into your network, using the IP address you entered in the `hercules.cnf` file. This might disconnect you when it sets up the network interfaces. If that happens, log in using the hercules prompt (remember to use '.' to interface with the guest OS) and shut it down nicely.

Now comes the host side configuration. If you've ever worked with network bridges and TAP/TUN devices in linux then this should be a breeze. If not, you should be ok with just copying everything I've done with only minor changes...

First, install the necessary utilities:

	sudo apt-get install uml-utilities bridge-utils

The following script is what I use to set up the bridge.

{% highlight shell %}
#IP of the host machine
HOST_IP=172.29.9.144
#IP of the zLinux installation
GUEST_IP=172.29.9.148
#Interface the host is using to connect to the internet
HOST_INTERFACE=eth0

#prepare eth0 for linking into the bridge
ifconfig $HOST_INTERFACE 0.0.0.0 promisc up

#setup bridge
brctl addbr bridge
brctl setfd bridge 0
brctl sethello bridge 0
brctl stp bridge 0
ifconfig bridge $HOST_IP netmask 255.255.0.0 up

#add eth0 to the bridge
brctl addif bridge $HOST_INTERFACE

#remove route added by hercules
route del -net $GUEST_IP netmask 255.255.255.255 dev tap0

#change tap0 hw address (it's the same address
#on the hercules side by default, so this needs to be changed)
ifconfig tap0 down
ip link set dev tap0 address 3E:31:C5:59:36:DD
ifconfig tap0 up

#bring tap up in promiscuous mode
ifconfig tap0 0.0.0.0 multicast promisc up

#add tap into the bridge
brctl addif bridge tap0
{% endhighlight %}

The script is pretty straight forward, but there are some important things to note. You can see that the script changes the `tap0` hardware address to something different than the interface that's defined in the `hercules.cnf` file. This is by design, and is required in order for packets to get forwarded correctly.

Additionally, this script must be run once the guest OS has __fully booted__. This poses an interesting problem, and you can either get around it using a lengthy sleep in your startup script, or by having the guest run the script on the host remotely using a passwordless ssh setup over the Channel to Channel connection.

That should be it!

Automatic startup hacks
----------------------

Right now, you still need to start up the bridge once the guest OS has initialized the network device. This is hardly optimal so I'm going to suggest a different solution.

First set up passwordless login to the host from the guest. Run the following as root.

	# ssh-keygen -t rsa
	# ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.1.1.3

Then `ssh root@10.1.1.3` to make sure you can login without a password.

Create a new file `/etc/init.d/bridge` with the following contents.

{% highlight shell %}
#!/bin/sh
echo "(re)Initializing Network Bridge"
/usr/bin/ssh root@10.1.1.3 sh /srv/zLinux_SUSE_11/bridge.sh
{% endhighlight %}

And symlink it to `rc3.d` right after the network script.

	# ln -s /etc/init.d/bridge /etc/init.d/rc3.d/S03bridge

And that's pretty much it. Once the network device has been initialized by the network startup script (`S02network` on my machine) it will ssh to the host and setup the bridge.

In order to start up hercules without having to log in (eg, from a startup script on the host) you can use the following script. It's not perfect, and can certainly be improved, but it works well enough for starting the vm from a startup script.

{% highlight bash %}
#start herc in a screen. use 'ipl 0120' at the prompt to boot the system
cd /srv/zLinux_SUSE_11/
screen -S 'fiery' -d -m sudo hercules -f /srv/zLinux_SUSE_11/hercules.cnf
echo "Hercules has started."
cd -
{% endhighlight %}

This should probably still be run as root, even though the sudo command is there. Otherwise the screen will start as the wrong user, and probably ask for a password too.

