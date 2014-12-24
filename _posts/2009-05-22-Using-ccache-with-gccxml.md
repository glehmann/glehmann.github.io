---
title: Using ccache with gccxml
---

I've patched [gccxml] to be able to use it with [ccache]. It was pretty simple — it was only necessary to:

* add a `-c` option which does nothing, but make [ccache] think that [gccxml] is not used to link some code, even if it makes no sense for [gccxml]. Without that option, [ccache] doesn't cache gccxml and log it in “called for link” — visible with `ccache -s`.
* add a `-o` option which does the same thing than the usual `-fxml=`, but in a more usual way for a compiler.
* and don't use the `--gccxml-gcc-options` option, as [ccache] doesn't like the commands with multiple input files. [ccache] reports it in its stats in the “multiple source files” section.

The results are quite interesting:

* gccxml without ccache: 2.532s
* gccxml + ccache, 1st run: 3.012s
* gccxml + ccache, next run: 0.436s

Brad King has just [accepted the patch][gccxml ccache], so it will be available in the next [CableSwig] release :-)

[WrapITK] is [patched already][wrapitk ccache] to support this feature — I hope to get quite impressive results on rebuild. I'll try to make some measures soon.

[WrapITK]: http://code.google.com/p/wrapitk
[gccxml]: http://www.gccxml.org/HTML/Index.html
[ccache]: http://ccache.samba.org/
[wrapitk ccache]: http://code.google.com/p/wrapitk/source/detail?r=357
[CableSwig]: http://www.itk.org/ITK/resources/CableSwig.html
[gccxml ccache]: http://public.kitware.com/cgi-bin/viewcvs.cgi/GCC_XML/GXFront/gxConfiguration.cxx?root=gccxml&r1=1.65&r2=1.66&sortby=date

