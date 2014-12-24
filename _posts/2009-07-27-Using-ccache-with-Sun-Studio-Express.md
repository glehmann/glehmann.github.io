---
title: Using ccache with Sun Studio Express
---

[Sun Studio Express] is _the_ compiler to use on OpenSolaris, at least because it's the only compiler usable to link C++ code to system provided C++ libraries. Ok. But unfortunately, it produce some "ccache internal error" when use with [ccache], and that's not very convenient for testing.

It is usable with [ccache-swig] though, the modified ccache provided with swig, as long as you define two environment variables:

    export CCACHE_CPP2=1
    export CCACHE_STRIPC=1

And that's all. Results for WrapITK: first build, 65m2.664s minutes; second build, 4m38.950s.

I'll try to package ccache-swig for OpenSolaris in the next days, patched to make the use of those variables useless.


[Sun Studio Express]: http://developers.sun.com/sunstudio/downloads/express/index.jsp
[ccache]: http://ccache.samba.org/
[ccache-swig]: http://www.swig.org/Doc1.3/SWIGDocumentation.html#CCache
