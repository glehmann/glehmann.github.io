---
title: Removing bzip2 dependency in boost
---

I had this error during [boost] build yesterday:

~~~
gcc.compile.c++ bin.v2/libs/iostreams/build/gcc-4.8/release/link-static/threading-multi/bzip2.o
libs/iostreams/src/bzip2.cpp:20:56: fatal error: bzlib.h: No such file or directory
 #include "bzlib.h"  // Julian Seward's "bzip.h" header.
                                                        ^
compilation terminated.
~~~

I don't need any bzip2 related feature for my project, and the [iostreams documentation]
says I can disable the bzip2 support with the `NO_BZIP2` Boost.build variable,
but it doesn't give any example of how to use it.

So here is how to use it: with the `-s` flag. For example:

    ./b2 -s NO_BZIP2=1

[boost]: http://www.boost.org
[iostreams documentation]: http://www.boost.org/doc/libs/1_57_0/libs/iostreams/doc/index.html?path=7