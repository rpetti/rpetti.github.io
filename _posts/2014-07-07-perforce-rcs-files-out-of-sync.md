---
layout: post
title: Perforce RCS files out of sync
---
During a server move relatively recently, we decided to go about it by setting up a replica in the new site, then switching it to a master once everything has been replicated over.

It was a pretty solid plan. Unfortunately, something went wrong and the replica was missing revisions from it's versioned files. This resulted in librarian checkout errors when trying to sync the missing revisions.

I still had the files from the original server, which contained the missing revisions, but unfortunately work had continued on many of the affected files, so we weren't able to copy them over verbatim. I searched around, but to my surprise there wasn't a utility for recombining revisions for RCS files. Perforce support also didn't have one, again to my surprise.

I wrote a small utility [here](https://github.com/rpetti/rcs-combiner) that recombines versioned files. It uses rcs to do much of the work, so that will need to be installed on whatever system you are using. The rcs-combiner acts on individual files, while the merge-p4-roots.sh script is a simple script that combines two perforce roots together.

Rather than running it against full p4 roots, I advise pulling out the specific versioned files you need to combine by examining the output of `p4 verify -q`, and piping the missing file#revisions into `p4 -x- fstat -Oc` to determine it's lbrFile (also known as the versioned/RCS file). As always, YMMV, and I take no responsibility if you try to run this against a live server and end up breaking something.

