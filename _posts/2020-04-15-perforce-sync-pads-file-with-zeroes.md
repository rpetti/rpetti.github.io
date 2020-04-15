---
title: "Perforce sync padding file with zeroes"
layout: post
---

I encountered an issue with a Windows build machine padding an otherwise correctly synced file with zeroes.

It seems that this is the "correct" behavior for some P4 clients when the metadata reports a different file size from the backing LBR file. After the LBR contents have been sent to the client, the client will then pad with zeroes to what it believes is the correct length. This can happen 
for a couple of reasons:

1. The LBR file is corrupt
2. The metadata has been modified
3. RCS keywords have changed in length for one reason or another

For reasons I cannot fathom, all of Perforce's metadata and checksums are performed on files _after_ RCS keywords have been replaced. This means that if those keyword values are changed for any reason (depot rename, user rename, etc) this can result in the files reported size being different from the actual.

In our case, this was caused by a user rename. Despite this being a fully supported operation, it regularly leaves your files in a possibly "invalid" state. I fixed this situation by using verify to recalculate the checksums/filesizes:

```
p4 verify -v //path/to/problem/file
```

Note that you should only do this if you are sure that the file has not been corrupted.
