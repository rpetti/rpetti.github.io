---
title: "Perforce oddities: \"Invalid date '-1'\""
layout: "post"
---

Recently a user had some issues submitting files to perforce. When they'd do so, they'd get the following error:

```
Librarian checkin ... failed.
Invalid date '-1'.
```

Apparently this is caused by the client not sending the server a valid file modification time. This can happen if the date is out of the range that can be expressed by a unix timestamp (1901-2038) or, as in this case, the client version is far too old.

As always, remember to keep your clients up to date. Backwards compatibility in p4 is fairly good, but only to a point, so best practice (IMO) is still to use the same version of the client as what is running on the server.