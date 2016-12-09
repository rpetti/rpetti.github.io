---
layout: post
title: 'Perforce Oddities: "already opened on this client"'
---
One of my users recently ran into a situation in which they couldn't open a file for editing because perforce thought the file was already opened on their client:

	$ p4 edit <somefile>
	<somefile> - can't edit (already opened on this client)

The bizzare thing is was that `p4 opened -a -u <user>` did not show this file, and the user wasn't able to revert it either. Running `p4 fstat` on the file, however, did show that the file was open on that client:

	$ p4 fstat <file>
	...
	... haveRev 1
	... ... otherOpen0 <user>@<client>
	... ... otherAction0 edit
	... ... otherChange0 default
	... ... otherOpen 1

Perforce support informed me that the issue was due to an inconsistency between database tables db.working and db.locks. `p4d -xx` doesn't seem to check for this, so you need to use another p4d command: `p4d -xf 925`

This can be run on a live master server without any issues. It ran very quickly in my case, and even wrote the changes to the journal so replicas will be able to pick them up without any manual intervention:

	$ p4d -r <P4ROOT> -xf 925

