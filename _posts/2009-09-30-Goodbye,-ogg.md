---
title: Goodbye, ogg
---

Finally, I give up. I've been using ogg to store most of my music for several years, but I'm tired of not being able to simply read my music somewhere else than on my computer. There are so much player everywhere! In the car, in my pocket, in my phone, … MP3 is too well established to do without it.

So, to make it simple, I convert all the ogg I have in mp3. I'll reextract the mp3 properly later, when time permit. I found a nice and simple tool for this task: [ogg2mp3]. With a command like this one

    find . -name *.ogg -print0 | time xargs -P8 -0 -l /tmp/ogg2mp3 -f --enc_opts --preset standard -S -- --dec_opts -Q --

a little more than 1000 ogg are converted in mp3 in 40 minutes on a 8 core nehalem 2.96 GHz. The hardest was to make the decision…

[ogg2mp3]: http://marginalhacks.com/bin/ogg2mp3
