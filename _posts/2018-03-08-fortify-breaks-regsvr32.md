---
title: "Fortify breaks DLL/OCX registration with regsvr32"
layout: post
---

HPE's Fortify security scanning solution has always been a pain to deal with, but this takes it to a new level. When running our builds with their msbuild wrapper, we encountered this error:

```
MSB8011: Failed to register output. Please try enabling Per-user Redirection or register the component from a command prompt with elevated permissions. 
```

Perhaps unsurprisingly, the recommendations in the error message did not help at all. Additionally, the build works perfectly fine without using the Fortify wrapper.

Fortify's support team was entirely useless with tracking down the issue, so we had to dive into it using Process Monitor to see what was happening.

When regsvr32 is executed by Fortify-wrapped builds, it doesn't check the current working directory (which is set to the project folder) for the referenced ocx. I have no idea why this happens. Nearly everything between both the wrapped and unwrapped regsvr32 executions are identical, but for some reason the wrapped one doesn't bother checking it's current working directory, while the unwrapped one does. Fortify support doesn't acknowledge this as the problem even in the face of direct evidence.

We did come up with a couple work-arounds, which may or may not work for your use-case:

1. Add the project directory to the PATH variable prior to calling your Fortify-wrapped msbuild command
    - This should allow regsvr32 to find the ocx/dll, as it still checks the entire PATH
    - Might not be viable if you have a multi-project solution
2. Contrive a means of having regsvr32 execute outside of Fortify's process tree
    - I don't really recommend this option because of how ridiculous it is, but it basically involves writing a program that masquerades as regsvr32, and uses a client-server setup to execute the real regsvr32 on a forked daemon. It's overkill, but we tested it and it works for us. You can find it [here](https://github.com/rpetti/command-decoupler).

As a side note, Fortify is a royal pain to integrate your builds with, and requires incredible amounts of resources even for a modest project. A 40 minute build can get as slow as 8 hours, and that's just for the translation phase. The analysis phase can take literally days to run, and I've seen it require 32GB or RAM or more.

If you are a devops or build engineer currently evaluating Fortify, I strongly recommend finding another option.