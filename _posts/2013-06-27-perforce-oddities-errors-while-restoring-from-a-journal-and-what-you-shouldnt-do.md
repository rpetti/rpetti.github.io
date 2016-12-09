---
layout: post
title: 'Perforce Oddities: errors while restoring from a journal, and what you shouldn''t
---
Twice this month I've run into the situation where one of our db.* files got corrupted, and started throwing "BTree is corrupt!" errors on various commands. The first instance was caused by our system running out of main memory, and reaping p4d processes while they were writing to the database tables. The second instance was caused by exhausted disk space.

Recovery was fairly simple in the first instance. I simply recovered from the last checkpoint, and replayed the journal overtop as per the documentation:
    
    $ p4d -r
    <p4root> -z -jr checkpoint.gz
    $ p4d -r
    <p4root> -jr journal

In the second instance, however, p4d threw errors while replaying the journal. This was likely due to the journal itself being truncated because of the lack of disk space. While I wasn't in the state of mind to record the specific error, the message claimed that I should try re-running the recovery command with '-f' to get past the error.

Do. Not. Do. This.

Perforce doesn't do any sort of check to see if a journal entry has already been replayed (possibly a technical limitation of the journal data model) so when you replay a journal that has already been replayed, weird things start happening...

In my case, the change counter was set to a value that was lower than the last change in the depot. This causes a "Sequence error: local change vs remote" error when checking in pending changesets. It was easily remedied by setting the change counter back to it's correct value, but unfortunately perforce __still tries to check the changeset in, even though there is clearly an issue__. For the life of me I can't understand why perforce would try to proceed when the server is clearly in an inconsistent state, but there you have it.

As a result, I had several changesets that got 'merged' together. Their descriptions would be changed to the new checkin description, and the file revisions were added to the existing changelist. While this didn't cause any data loss, it causes a lot of confusion for people examining the history, and as near as I can tell there's no way to fix it without somehow resequencing the changesets in the original journal.

