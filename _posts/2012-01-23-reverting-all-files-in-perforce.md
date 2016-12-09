---
layout: post
title: Reverting All Files In Perforce
---
I often find myself in the situation where I need to delete the perforce account of a user who has left the company. Unfortunately I'm usually unable to do so because they have left files open, resulting in this error:

	$ p4 user -d -f SOMEUSER
	User SOMEUSER has file(s) open on 2 client(s) and can't be deleted.

The solutions posted online usually involve either deleting the clients, or 'masquerading' as the client host and reverting the files by hand. This is very tedious, so I wrote a python script to do it instead. It's a bit crude, but all it does is take usernames off the command line, and proceeds to revert all the files that user has open on the server.


{% highlight python %}
#!/usr/bin/python

import marshal
import subprocess, shlex
import re
import os
import sys

def getPerforceResponse(command):
	lst = []
	args = shlex.split(command)
	p = subprocess.Popen(args, stdout=subprocess.PIPE)
	try:
		while 1:
			dictionary = marshal.load(p.stdout)
			lst.append(dictionary)
	except EOFError:
		pass
	return lst

def getOpenedFiles():
	return getPerforceResponse("p4 -G opened -a //...")

def getOpenedFilesForUser(userID, opened):
	openedByUser = []
	for i in opened:
		if i['user'] == userID:
			openedByUser.append(i)
	return openedByUser

def getClient(clientID):
	return getPerforceResponse("p4 -G client -o \"" + clientID + "\"")[0]

def main():
	print "Getting list of open files..."
	opened = getOpenedFiles()
	for user in sys.argv[1:]:
		openedByUser = getOpenedFilesForUser(user, opened)
		print "Found " + str(len(openedByUser)) + " opened files for " + user
		print "reverting..."
		for f in openedByUser:
			print "Reverting " + f['depotFile'] + " on " + f['client']
			client = getClient(f['client'])
			host = client.get('Host', 'p4client')
			response = getPerforceResponse("p4 -G -H " + host + " -c " + f['client'] + " -u " + user + " revert -k \"" + f['depotFile'] +"\"")
			print response

main()
{% endhighlight %}

