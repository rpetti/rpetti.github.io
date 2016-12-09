---
layout: post
title: Adding a Windows slave to Jenkins the hard way
---
Jenkins makes adding slaves really easy. For unix machines, the preferred method is to use SSHD, which only requires that the master be able to contact the slave. On windows, however, you are stuck using a JNLP based solution, which requires that the slave be able to access master over the network.

I got stuck in the awkward situation where I had to add a windows machine to our master, but the slave could not even ping it. This could be resolved by updating our routing tables on the slave side, but unfortunately that wasn't an option here. Instead I chose to try and start the slave using a Windows SSHD solution.

**Edit:** __I've found that the data throughput on FreeSSHD using the method I've outlined below leaves much to be desired. I've since switched over to installing cygwin and using it's sshd server instead. If you are using this pipe to send or archive files, I highly recommend using cygwin's sshd server instead, as it's several orders of magnitude faster. For example, I went from a 45 minute transfer, down to only 1 minute.__

First things first, we need to install an ssh server onto the master. This was pretty easy, as I just used [FreeSSHD](http://www.freesshd.com/). Once installed, you need to make sure to stop the service, and run it by hand as administrator in order to change the settings. It'll show up in the toolbox on your system, so just right click it and click settings. From there you can set up your new user.

A few gotchas: You'll want to DISABLE "Use new console engine" on the SSH tab. Also, when setting up pubkey authentication, you'll want to put the key into a file in the "Pubkey folder" that's defined in the Authentication tab of FreeSSHD. The file needs to have the exact same name as the new user you created. In my case, my user was called "releng" so I made a new text file called "releng" (no extension!) with the contents of my jenkins user's public key.

Test the connection from the master user by sshing into the slave as the jenkins user:

	jenkins@jenkins-master:~$ ssh releng@windows-slave-hostname

Once you've verified that passwordless authentication works, now it's time to set up the slave. Copy the slave.jar from your jenkins install (http://yourjenkinsurl/jnlpJars/slave.jar) onto the slave somewhere. In this example, I'll assume you placed it into `C:\jenkins\`.

Give it a shot again:

    jenkins@jenkins-master:~$ ssh releng@windows-slave-hostname java -jar C:/jenkins/slave.jar -text

It should return something like:

    
    <===[JENKINS REMOTING CAPACITY]===>rO0ABXNyABpodWRzb24ucmVtb3RpbmcuQ2FwYWJpbGl0eQAAAAAAAAABAgABSgAEbWFza3hwAAAAAAAAAAY=<===[HUDSON TRANSMISSION BEGINS]===>rO0ABQ==

That means it's working, but there's still more work to be done to get jenkins talking to it properly. First, set up a new Dumb slave in jenkins like you normally would, and select "Launch slave via execution of command on the Master". Unfortunately the built-in ssh functionality doesn't work because of the way FreeSSHD handles stdout and stderr.

Here's the launch command you need it to run:
    
    bash -c 'ssh releng@windows-slave cmd /c "java -jar C:/jenkins/slave.jar -text 2>C:/jenkins/slave.junk.txt"'

The redirect to slave junk is necessary because the slave.jar outputs informational messages to it's stderr. Normally these messages are not sent back to the master, but in this case FreeSSHD just pipes stderr into stdout, which would cause problems with the master as it parses the output. I'm not 100% sure if wrapping it in bash is necessary, but it seems to work fine.

Good luck!

