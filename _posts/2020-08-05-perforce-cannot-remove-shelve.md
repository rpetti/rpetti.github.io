---
title: "Perforce cannot remove shelved changelist: 'needs tofile'"
layout: "post"
tags: perforce p4
---

After recently upgrading to 2020.1, we began encountering users that have somehow created broken shelved changelists.

```
p4 shelve -df -c 29074337
//my/original/file - needs tofile //my/new/file
Shelve aborted -- fix problems then use 'p4 shelve -c 29074337'
```

Essentially the user has somehow shelved one-half of a move file change. Normally this should be impossible, but it seems there was a regression that started to allow this to happen. The only way to remove such a shelved change is by manually modifying the database, so I'll try to walk through it here. **As always, take a backup before doing any sort of database modification!**

Steps:

1. Take a checkpoint if you don't have one already
2. Grep the checkpoint for the change in question and save it to a file:

```
zgrep 29074337 checkpoint.gz | tee 29074337.jnl
```

3. Edit the new file, making the following changes:
  - delete any lines that aren't for the db.change, db.changex, or db.workingx tables
  - modify the db.changex and db.change rows to be a replace value operation `@rv@` instead of `@pv@`
  - modify the 8th column of the db.changex and db.change rows to be `0` instead of `2`
  - modify the db.workingx rows to by a delete value operation `@dv@` instead of `@pv@`
  - review the rows to ensure they aren't making changes to unintended changelists

Here's the schema documentation if you want a better understanding of the fields being modified: [https://www.perforce.com/perforce/doc.current/schema/](https://www.perforce.com/perforce/doc.current/schema/)

example:

```
@rv@ 6 @db.change@ 29074337 29074337 @myworkspace@ @myuser@ 1590408588 0 @My Description@ @@ @@ @@ 1593587390 1593070995 @@
@rv@ 6 @db.changex@ 29074337 29074337 @myworkspace@ @myuser@ 1590408588 0 @My Description@ @@ @@ @@ 1593587390 1593070995 @@
@dv@ 10 @db.workingx@ @//29074337/my/new/file@ @//my/new/file@ @29074337@ @myuser@ 3 3 0 32 7 29074337 0 0 00000000000000000000000000000000 -1 0 0 32 @//my/original/file@ 0
```

4. Restore the modified checkpoint file:

```
p4d -r $P4ROOT -s -jr 29074337.jnl.modified
```

Note the `-s`, this restores the checkpoint and applies the changes to your running journal, allowing the changes to be automatically propagated to your replicas and offline checkpoints. You can run this command against a live server without issues.