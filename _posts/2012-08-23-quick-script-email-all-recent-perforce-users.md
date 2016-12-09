---
layout: post
title: 'Quick Script: Email all recent Perforce users'
---
Here's a script I whipped up in order to send an email to all recent Perforce users. I needed this because my company uses a shared license, so all user accounts are shared between our perforce servers. When a server needs to go down for maintenance, I like to email only those people who actually use it. This uses python, but it does not require the p4python api (though I imagine it would be quite simpler using it). I'm sure there are some imports that need to be cleaned up, but whatever, it's just a quickie.

{% highlight python %}
#!/usr/bin/python

import marshal
import subprocess, shlex
import re
import os
from datetime import datetime
import time
import smtplib
from email.mime.text import MIMEText

today = int(time.time())

window = 60*60*24*30 #30 days

mailhost = "mailhost"
me = "you@yoursite.com"

maxmails = 10

msg = MIMEText("Perforce server is going down! Lookout!")
msg['Subject'] = "Perforce downtime this weekend"
msg['From'] = me

def getRecentUsersList():
    args = shlex.split("p4 -G users")
    p = subprocess.Popen(args, stdout=subprocess.PIPE)
    userList = []
    while True:
        try:
            user = marshal.load(p.stdout)
            if int(user['Access']) > (today-window):
                userList.append(user)
        except:
            break
    return userList

def getUniqueEmails(userList):
    emails = {}
    for user in userList:
        emails[user['Email']] = 1
    print "got " + str(len(emails.keys())) + " emails"
    return emails.keys()

def sendEmail(emails):
    print "sending email to "+str(emails)
    s = smtplib.SMTP(mailhost)
    s.sendmail(me, emails, msg.as_string())
    s.quit()

def sendEmails(emails):
    emailsubset = []
    for email in emails:
        emailsubset.append(email)
        if len(emailsubset) >= maxmails:
            sendEmail(emailsubset)
            emailsubset = []
    if len(emailsubset) > 0:
        sendEmail(emailsubset)

def main():
    userList = getRecentUsersList()
    emails = getUniqueEmails(userList)
    sendEmails(emails)

main()
{% endhighlight %}

