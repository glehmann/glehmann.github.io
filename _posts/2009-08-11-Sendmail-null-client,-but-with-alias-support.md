---
title: Sendmail null client, but with alias support
---

In our research lab, all the users have the same login on all the servers. We also have a single mail server used by all the users in the research lab. In that case, the natural way of managing the mails sent from the different servers seems to send them to the main mail server, so the users only have a single place to check there mails. There is an exception though: the administrative users, like root, are not the same real users on all the servers, so each system must be able to apply the aliases to send the administrative mail to the right users.

I think the case is very common, but it must be not that well documented, because I haven't been able to find a clear example for sendmail. Here is the configuration I used — I hope it will be useful for someone!

    OSTYPE(`solaris8')
    define(`MAIL_HUB', `smtp.jouy.inra.fr')
    define(`SMART_HOST', `smtp.jouy.inra.fr')
    MASQUERADE_AS(`jouy.inra.fr')
    FEATURE(`allmasquerade')
    FEATURE(`masquerade_envelope')
    MAILER(`local')
    MAILER(`smtp')
