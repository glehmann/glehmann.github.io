---
title: Can ccache speed up wrapitk build?
---

[WrapITK] is soooooo long to build. There are several things we can do to speedup things a bit. The best one, IMO, would be to explicitly instantiate all the type used in [WrapITK] and group them in a library. That way, the compiler wouldn't spend so much time compiling and compiling the base classes again. Sadely, the explicit instantiation stuff in ITK never shown any improvements…

The other option which may be interesting is [ccache]. [WrapITK] is made in such a way that a simple change in a file can trigger the recompilation of many files. This is a required behavior, because that changes can affect the content of the newly compiled files, but most of the time, it doesn't change it, and the compiler is working hard again to produce the exact same thing. Maybe [ccache] can help in that case. Another use case where it may be useful is to rebuild the tests, in a daily test procedure.

Ok, so, I've tried it:

    [glehmann@pcconf5 build-ccache]$ CC="ccache gcc" CXX="ccache g++" ccmake ..
    [glehmann@pcconf5 build-ccache]$ make
    ...
    1680.23user 250.11system 34:10.61elapsed 94%CPU (0avgtext+0avgdata 0maxresident)k
    69056inputs+3063824outputs (92major+23606438minor)pagefaults 0swaps
    [glehmann@pcconf5 build-ccache]$ make clean
    [glehmann@pcconf5 build-ccache]$ make
    460.59user 110.94system 10:04.63elapsed 94%CPU (0avgtext+0avgdata 0maxresident)k
    0inputs+2086472outputs (0major+8874841minor)pagefaults 0swaps


Great! I wonder why I haven't done that before…

[WrapITK]: http://code.google.com/p/wrapitk
[ccache]: http://ccache.samba.org/
