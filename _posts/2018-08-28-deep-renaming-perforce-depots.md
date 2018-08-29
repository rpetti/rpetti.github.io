---
title: "(Deep) Renaming Perforce Depots"
layout: post
---

If you have ever tried to rename anything in Perforce, you know that it's rarely an easy thing to do. Depots especially are practically impossible to rename without crippling any current development.

You can find Perforce's official documentation on how to "rename" a depot [here](https://community.perforce.com/s/article/5427). However, if you were to follow their documentation, a _ton_ of things will break including clients, branches, labels, shelved changes,protection tables and triggers. It's a no-go for _any_ production environment!

When pressed for a method of performing a deep rename that would change the database itself, Perforce Support told us we would need to hire contractors to do it for us. This was despite us already paying for a support contract, and them being aware of the need for such a feature for over 11 years!

So, I wrote my own damned renamer. You can find it on my [GitHub](https://github.com/rpetti/p4-depot-renamer).

It's fairly easy to use, but requires downtime. It renames the depot by reading checkpoint entries line by line and renaming the specified depot in the relevant areas, producing a new checkpoint file that you can restore from. It's configured according to the database spec Perforce published for 2018.1, and I've tested it against the same.

As always, take backups, and your mileage may vary. While I don't take responsiblity if something goes wrong, feel free to hit me up if you find any problems with it!