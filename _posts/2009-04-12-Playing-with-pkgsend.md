---
title: Playing with pkgsend
---

I was used to contribute some packages in [Mandriva Linux], so contributing packages is quite natural for me, even if I haven't done it for some time, for various reasons. OpenSolaris has its own packaging system — pkg(5). Today, I've tried to play a bit with it, to create a very simple package with one of my very simple program: [abspath].

pkg(5) is only involved in the package distribution — unlike rpm and other packaging systems, it has nothing to do with the build of the packaged program. To make the packages, it only provides a simple program used to fill the repository: `pkgsend`. A reproducible way to build the package looks like an important feature though, so I'm not sure this is the best approach. By the way, the contributions to OpenSolaris (through [SourceJuicer]) are made with a rpm-like spec file processed by [pkgbuild], so It seems I'm not the only one to have some doubts on the approach. At least it provides a separation of build and package upload which allow to create different tools to make packages. Anyway, I've first tested `pkgsend`. I'll try `pkgbuild` later.

First, I've configured a local pkg server to have somewhere to put my new package. I think that's possible to use a simple file repository, but the pkg server is much documented, and is the way to go to upload to a remote repository. Enabling the pkg server should be only the two last lines of the following commands. I'm not sure why, but the default pkg server port is 80, which is not really convenient, especially when `pkgsend` uses 10000 as default. I changed it to 10000.

    svccfg -s pkg/server:default addpg pkg application
    svccfg -s pkg/server:default setprop pkg/port = 10000
    svcadm refresh pkg/server:default
    svcadm enable pkg/server:default
    pkg set-authority -O http://localhost:10000 local-test

I've then created a small manifest — `manifest.pkg` — to describe the `abspath` package, its content, its dependencies, its author, etc. Here it is:

    set name="pkg.name"        value="abspath" 
    set name="pkg.description" value="Return the absolute path of a file or directory." 
    set name="maintainer"      value="Gaetan Lehmann <gaetan.lehmann@jouy.inra.fr>" 
    set name="upstream"        value="Gaetan Lehmann <gaetan.lehmann@jouy.inra.fr>"
    depend type=require fmri=SUNWPython
    file abspath mode=0555 owner=root group=bin path=/usr/bin/abspath 

Only a single file. The package should also contain a man page, but it must be compiled with `asciidoc` which is not packaged for OpenSolaris, so I've done without it for now.

`pkgsend` is then used to upload the package to the repository. No need to give the repository to `pkgsend`: the default one is the right one.

    eval `pkgsend open abspath@0.1`
    pkgsend include manifest.pkg
    pkgsend close

I understand the interest of the `open` … `close` subcommands when calling `pkgsend add`, `pkgsend set`, etc. manually, but it's a bit strange to have to use those three commands with a manifest. I would have been much pleased by something like 

    pkgsend commit abspath@0.1 manifest.pkg

It would also avoid the not so natural use of `eval`. Maybe that something we'll see in the future…

Once the package sent to the pkg server, it can be installed with `pkg install abspath` and uninstalled properly with `pkg uninstall abspath`, and the repository can directly be used on other computer as well — quite nice.

[Mandriva Linux]: http://www.mandriva.com
[abspath]: http://voxel.jouy.inra.fr/darcs/abspath
[SourceJuicer]: http://opensolaris.org/os/project/sourcejuicer/
[pkgbuild]: http://pkgbuild.sourceforge.net/pkgbuild-ips.php

