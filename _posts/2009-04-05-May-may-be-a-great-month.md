---
title: May may be a great month
---
It's been a long time since my last blog entry. I began the blog, and, well, didn't thought
to write my life in it. I'll try to be a little more productive in the next days.

One of the project of my team has been accepted: I'll have to set up a server to run an
[Open Microscopy Environment][OME] ([OME]) service. We got the funds with that projects for:

* two storage servers;
* a workstation;
* a macbook air.

Everything should be there next month — a great month, on work side :-)

I've decided to go with [Solaris 10] for the servers. The main feature which leaded me
to choose Solaris for that project is the [ZFS] compression — the preliminary
testings have shown a compression rate of 2.7 on our data. That's a very nice increase
of available space. Just imagine that a storage pool of 10 TB can actually store 27 TB
of data…

I've been googling a lot these last months, to learn more on Solaris and [Opensolaris],
and I must say I'm quite pleased by most of the things I see. As a consequence, and after
having checked that my main tools ([ITK] and [WrapITK]) are usable, I've decided to
go with OpenSolaris on the workstation, instead of the [Mandriva Linux] I was used to use.
It doesn't mean I'll stop using Mandriva: I still use it at home, and the tools I'm
developing are installed on several hosts with that system at work.
Also, I know that OpenSolaris is not yet well polished up to now, especially the command line
interface. It will surely need a bit of work to make it as user friendly as linux, but I
think it's worth the cost.

I'll try to blog in the next days on OME, Solaris and OpenSolaris.

[OME]: http://www.openmicroscopy.org
[Solaris 10]: http://www.sun.com/software/solaris/10/index.jsp
[ZFS]: http://opensolaris.org/os/community/zfs
[OpenSolaris]: http://www.opensolaris.org
[Mandriva Linux]: http://www.mandriva.org
[ITK]: http://www.itk.org
[WrapITK]: http://code.google.com/p/wrapitk

