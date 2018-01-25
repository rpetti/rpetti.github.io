---
layout: post
title: Reverting All Files In Perforce
---
I often find myself in the situation where I need to delete the perforce account of a user who has left the company. Unfortunately I'm usually unable to do so because they have left files open, resulting in this error:

	$ p4 user -d -f SOMEUSER
	User SOMEUSER has file(s) open on 2 client(s) and can't be deleted.

The solutions posted online usually involve either deleting the clients, or 'masquerading' as the client host and reverting the files by hand. This is very tedious, so I wrote a python script to do it instead. It's a bit crude, but all it does is take usernames off the command line, and proceeds to revert all the files that user has open on the server.

<script src="https://gist.github.com/rpetti/f250fb699b37123ea8bb13617f121e11.js"></script>
