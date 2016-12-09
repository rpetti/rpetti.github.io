---
layout: post
title: Perfmerge++ Warnings are not actually warnings!
---
I recently did a migration of source code from one perforce server to another. During the merge process, I ran into an odd warning:
    
    perfmerge++ warning:
            No mapping for change 293174 in database /x/sourceserver/server/. Rejecting.

Thinking that it was just a dead or missing changeset, I skipped right along and proceeded to put the server into production. Verify came back ok, so I figured everything was fine.

It was not...

It seems that perfmerge has an annoying tendency to quit at the first sign of trouble, then throw a 'warning' rather than a full blown error message. In this case, it was about to write out the db.changes* tables, then decided against it. When I brought the server back online, there were file revisions, but no changesets or change descriptions to speak of. This is obviously a huge problem!

I checked both source p4 roots, but couldn't find the changes in question. Out of desperation, I took checkpoints of both roots, and grepped the entire checkpoint looking for the changeset numbers that were throwing warning. There were two things that I found.

The first thing was that somehow, someone had managed to open files under a pending changeset that didn't exist. I'm not entirely sure how that happened, but it did. Once that was found, it was a simple matter of reverting all the files opened under that missing changeset.

The second thing I found wrong was that perfmerge apparently had an issue with the way my job names were formatted, which caused any changeset tied to that job to fail to merge. In my case, we don't make heavy use of jobs, and the one with the issue appeared to be accidentally created. After removing the fix tying the changeset to it and deleting it, the merge proceeded to work fine.

Hopefully this helps some people. I regret that I didn't save my entire solution somewhere, but the perforce documentation should cover most of what's required to do the same.

