---
title: Install IHaskell on Mac OS X
---

[jupyter] notebooks are awesome, and there are many kernels available for our
favorite languages. I wanted to install [IHaskell] for some time, but always
failed either to build it altogether, or to make it work. I finally succeeded
today, and it wasn't that difficult!

The dependencies have been installed using [homebrew], and I already had a lot of
stuff installed that I believe are also needed (XCode command line tools,
xquartz, pango, cairo, python, jupyter, …)

I used stack to build and install IHaskell:

    brew install stack
    stack setup
    stack install ihaskell

Last step, install the jupyter kernel. `ihaskell` has a subcommand for that:

    ~/.local/bin/ihaskell install

It needs a little help though to work properly with `stack`. The kernel spec is installed
in `~/Library/Jupyter/kernels/haskell/kernel.json`. In order to run the kernel
in the stack environment, `stack exec --` must be added before the kernel
command in the `argv` entry:


    {
        "display_name": "Haskell", 
        "language": "haskell", 
        "argv": [
            "stack", "exec", "--", 
            "/Users/glehmann/.stack/snapshots/x86_64-osx/lts-6.0/7.10.3/bin/ihaskell", 
            "kernel", 
            "{connection_file}", 
            "--ghclib", 
            "/Users/glehmann/.stack/programs/x86_64-osx/ghc-7.10.3/lib/ghc-7.10.3"
        ]
    }

That's it! It is now possible to select `Haskell` when creating a new notebook
in jupyter!

[jupyter]: http://jupyter.org
[IHaskell]: https://github.com/gibiansky/IHaskell
[homebrew]: http://brew.sh/
