---
title: Slow git gerrit-push
---

I've been annoyed by the very slow process of submitting changes to [ITK] since
the beginning of the year. Each `git gerrit-push` took approximately 10 minutes
to run.

That would have annoyed me a lot more a few years when I was doing it several
times a day, but now that it is only a hobby, I just let it run while actually working.

But now in the train, with the internet access failing every minute or so, this
is quite annoying. So I finally took a few minutes to search on my favorite
search engine, and discovered that a simple

    GSSAPIAuthentication no

in my `~/.ssh/config` fixed the problem. `git gerrit-push` now runs in just a
few seconds. And this is in the train, not with a fast connection.

I should have looked at it a lot earlier!

[ITK]: http://review.source.kitware.com/#q,status%3Aopen+project%3AITK,n,z
