---
title: "Perforce Parallel Sync Causes Garbled Log Output"
layout: post
---

We recently upgraded our perforce client from 2015.1 to 2018.1, and immediately ran into a strange issue where running `p4 -s sync` with parallel syncing enabled would cause strange output.

Namely, when we run it, we get `exit: 0` interspersed with the regular output near the end of the log, often even breaking other log lines!

```
$ p4 -s sync --parallel=threads=5
...
info: //depot/some/fiexit :0
le.txt#1 - added as //workspace/depot/some/file.txt
exit: 0
exit: 0
exit: 0
exit: 0
```

It seems that at some point, the client was changed to print it's exit code for every thread instead of just the parent, and this in combination with the exit code output not being synchronized properly results in the log getting garbled near the end of the operation.

Perforce support claims that this has been an issue since 2014.1, when parallel operations were first introduced, but my tests show that it was introduced in 2016.2. They also claim that it's a very difficult fix, despite being an extremely obvious one (don't print exit codes when you are a child process/thread).

I've tried to explain this to them, but it seems Perforce support has taken a massive nosedive lately in terms of quality... In the meantime, their "recommendation" is to turn off parallel syncing entirely, or not rely on the log output from `-s`. Neither of these are real solutions, but it's what we're stuck with until Perforce gets their head out of their asses and actually listens to customer feedback again.