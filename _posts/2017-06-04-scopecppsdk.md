---
title: "What the heck is ScopeCppSDK?"
layout: post
---

When trying to upgrade our build infrastructure to use Microsoft's new Visual Studio 2017, I got a strange request from a developer: They needed include paths from an SDK called "ScopeCppSDK", which they claimed that this was installed with VS2017 within it's installation folder, and that this was the new location for some standard libraries.

So I checked my own system, and the folder was absent. I checked some of our build systems, and the folder was absent there as well. Since these were fairly common libs such as stdlib.h, it seemed strange that they weren't installed with the C/C++ compilers...

It turns out Microsoft did move them, just not to where the dev thought. Microsoft's [blog](https://blogs.msdn.microsoft.com/vcblog/2015/03/03/introducing-the-universal-crt/) goes into more detail, but the summary is that the new location for these UCRT headers and libraries is the Windows 10 SDK. JUST the Windows 10 SDK. Nowhere else.

So what the heck is ScopeCppSDK? After screwing around with the VS2017 installer, bisecting by adding and removing components, I found out that it's actually part of the Azure SDK. Since we aren't using Azure, I told the dev they were using the wrong location for those libs, and I was finally able to update our build environment scripts properly.

I do like some of the changes in VS2017, but between everything getting moved around and no longer being able to query the registry for installation information, it's been an exercise in frustration to get our build environment scripts updated accordingly.
