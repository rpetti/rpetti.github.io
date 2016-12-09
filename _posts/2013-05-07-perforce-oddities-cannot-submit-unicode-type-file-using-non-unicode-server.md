---
layout: post
title: "Perforce Oddities: cannot submit unicode type file using non-unicode server"
---
Ran into this little gem today... I've got a project that I'm currently trying to branch. It's a very simple integration operation from one location to another, nothing crazy. Of course Perforce needs to make it as complicated as possible by throwing me this error for a couple of our files:

`cannot submit unicode type file using non-unicode server`

Now, this is incredibly nonsensical. I'm integrating an __already submitted__ file from one location to another, and checking it in. Why does Perforce reject it if it's already been submitted successfully once before?

It turns out that the way perforce handles unicode filetypes changed at some point. Before this change, you could submit unicode files without any problems. After this change, your server needs to be reconfigured as a 'unicode server' in order to check in unicode files, even though it's clearly handling already checked-in files just fine. To me, this seems like an artificial restriction, but I digress.

To resolve this, you just need to reopen the files as text, and resubmit them.
    
    $ p4 reopen -t text <FILE>

After that, you can work on the files again as normal. I'm not sure what effect this has on the encoding of the files themselves, so as always, YMMV, but at least you'll be able to get on with your life.

