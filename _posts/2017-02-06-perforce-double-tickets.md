---
title: "p4 login creates two tickets when using a replica"
layout: post
---

I do a lot of Perforce automation, both manually using scripts, and using Jenkins plugins like the [p4-plugin](https://wiki.jenkins-ci.org/display/JENKINS/P4+Plugin) and [perforce-plugin](https://wiki.jenkins-ci.org/display/JENKINS/Perforce+Plugin), the latter of which I supported for many years. All of these methods (usually) use perforce tickets so it doesn't pass the password around plaintext.

Unfortunately, in some perforce setups, the command to get the tickets ```p4 login -p``` returns _more than one ticket_. What's even worse, is that each of these tickets appears to only work for specific commands. For example, the first might work fine for changing a client spec, and the second might work fine for fetching a client spec, but when you try to switch that around it says the password is invalid.

This was caused by how Perforce does authentication in a replicated environment by default. If you log into a replica, you get two tickets, one for the replica (which can only read) and one for the master (which can only write). This is even when the replica is configured to forward everything.

This is an _insane_ default, as it makes a replicated environment almost useless when using ticket automation. Fortunately, the solution is [here](http://answers.perforce.com/articles/KB/11958). In my case, I was running v15.1, so I just ran this:

	p4 configure set cluster.id=myClusterName
	p4 configure set replicaName#rpl.forward.login=1

Note that apparently these configurables have changed multiple times in the past few years, so check the KB so see what works for your version.

The one caveat is that changing these setting invalidates all current login tickets, including the service account ticket used for replication. After re-logging in, the replica started working again.
