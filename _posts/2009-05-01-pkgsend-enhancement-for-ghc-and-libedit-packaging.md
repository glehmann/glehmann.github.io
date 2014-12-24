---
title: pkgsend enhancement for ghc and libedit packaging
---

With a single file, `abspath`, was a program easy to package. Using `pkgsend` only to package bigger program would be quite painful: it would require to call `pkgsend file` for all the files of the program. While this is a nice practice to ensure the required files are there, it is also painful when one simply wants to quickly build a package for a big program.

To avoid that step, it would be possible to create a program which fills the manifest for the packager, and then `pkgsend include`, or create a program which runs `pkgsend file` for all the files, with the right attribute. While those two approaches seem valid, I don't like much the idea to have yet another program — it should be possible to do everything with `pkgsend`. 

`pkgsend` provides an `import` subcommand in charge of importing the content of a traditional package. `pkgsend` is coded in python, and is quite easy to modify. So I've patched it to add a `tree` subcommand, which import a whole tree in the package. I'm not sure `tree` is the right name — I've never been good at choosing names. Anyway, here's [the patch][pkgsend-tree.patch].

Ok, now back to my main interest: packaging [GHC]. GHC is tricky to build, because it requires GHC. So, before building it from source in the future, I've used the prebuilt binary provided on GHC website. It requires a library not available in IPS at this time: [libedit]. I began with this one. For a clean work, I think I'd have to provide both 32 and 64 bits binaries. I'm not sure yet how to do that, so I simply used the default build options.

    wget http://www.thrysoee.dk/editline/libedit-20090405-3.0.tar.gz
    gtar xvzf libedit-20090405-3.0.tar.gz
    cd libedit-20090405-3.0
    ./configure --prefix=/usr
    gmake install DESTDIR=/tmp/libedit

Then I created a very simple manifest:

    set name="pkg.name"        value="libedit"                                                                                                                               
    set name="pkg.description" value="Command line editor library"                                                                                                           
    set name="maintainer"      value="Gaetan Lehmann <gaetan.lehmann@jouy.inra.fr>"                                                                                          
    set name="upstream"        value="Jess Thrysoee"                                                                                                                         

Then I used `pkgsend` to actually create the package.

    eval `pkgsend open libedit@20090405-3.0`
    pkgsend include libedit.pkg
    pkgsend tree /tmp/libedit                                                                                                                                                        
    pkgsend close

Note the use of the new `tree` subcommand, which only takes the base dir as argument.

That's it. The package is now installable.

    pfexec pkg refresh
    pfexec pkg install libedit

The same with ghc

    wget http://haskell.org/ghc/dist/6.10.1/maeder/ghc-6.10.1-i386-unknown-solaris2.tar.bz2
    gtar xvjf ghc-6.10.1-i386-unknown-solaris2.tar.bz2
    cd ghc-6.10.1
    pfexec pkg install SUNWgnu-mp
    pfexec pkg install SUNWgcc
    pfexec pkg install SUNWgmake
    ./configure --prefix=/usr
  
There is a small thing to fix in the generated `Makefile` — not sure why. Here is the patch:

    --- Makefile~	2008-11-10 10:41:29.000000000 +0100
    +++ Makefile	2009-04-20 22:24:21.841372547 +0200
    @@ -32,7 +32,7 @@
      $(MAKE) -C gmp       install      DOING_BIN_DIST=YES
      $(MAKE) -C docs      install-docs DOING_BIN_DIST=YES
      $(MAKE) -C libraries/Cabal/doc install-docs DOING_BIN_DIST=YES
    -	$(INSTALL_DATA) $(INSTALL_OPTS) extra-gcc-opts $(libdir)
    +	$(INSTALL_DATA) $(INSTALL_OPTS) extra-gcc-opts $(DESTDIR)$(libdir)
    
      install :: postinstall denounce
   
Then install

    gmake install DESTDIR=/tmp/ghc/

Create the manifest:

    set name="pkg.name"        value="ghc"                                                                                                                                   
    set name="pkg.description" value="State-of-the-art, open source, compiler and interactive environment for the functional language Haskell"                               
    set name="maintainer"      value="Gaetan Lehmann <gaetan.lehmann@jouy.inra.fr>"                                                                                          
    depend type=require        fmri=SUNWgnu-mp                                                                                                                               
    depend type=require        fmri=SUNWgcc       
    depend type=require        fmri=SUNWncurses
    depend type=require        fmri=libedit                                                                                                                                  

Create the package:

    eval `pkgsend open ghc@6.10.1`
    pkgsend include ghc.pkg
    pkgsend tree /tmp/ghc                                                                                                                                                        
    pkgsend close


Note that to avoid the message

    pkgsend: 'add' failed for transaction ID '1241183161_pkg%3A%2Fghc%406.10.1%2C5.11%3A20090501T130601Z'; status '500': <urlopen error (12, 'Not enough space')>

I had to add more swap memory than the 512MB available by default:

    pfexec swap -d /dev/zvol/dsk/rpool/swap
    pfexec zfs destroy rpool/swap
    pfexec zfs create -V 4G rpool/swap
    pfexec swap -a /dev/zvol/dsk/rpool/swap

As for libedit, ghc can now be installed

    pfexec pkg refresh
    pfexec pkg install ghc

Next step, publish the packages to a public repository.

[GHC]: http://haskell.org/ghc/
[libedit]: http://www.thrysoee.dk/editline
[pkgsend-tree.patch]: http://glehmann.net/pkgsend-tree.patch
