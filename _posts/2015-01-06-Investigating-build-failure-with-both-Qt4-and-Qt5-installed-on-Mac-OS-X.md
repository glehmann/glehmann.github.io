---
title: Investigating build failure with both Qt4 and Qt5 installed on Mac OS X
---

I'm getting these errors while building Qt5 programs on my Mac:

~~~
In file included from /usr/local/opt/qt5/lib/QtWidgets.framework/Headers/QApplication:1:
In file included from /usr/local/opt/qt5/lib/QtWidgets.framework/Headers/qapplication.h:48:
In file included from /usr/local/opt/qt5/lib/QtGui.framework/Headers/qguiapplication.h:39:
/usr/local/opt/qt5/lib/QtGui.framework/Headers/qinputmethod.h:82:5: error: unknown type name 'QLocale'
    QLocale locale() const;
    ^
/usr/local/opt/qt5/lib/QtGui.framework/Headers/qinputmethod.h:91:21: error: no type named 'InputMethodQueries' in namespace 'Qt'; did you mean 'InputMethodQuery'?
    void update(Qt::InputMethodQueries queries);
                ~~~~^
/usr/local/include/QtCore/qnamespace.h:1541:10: note: 'InputMethodQuery' declared here
    enum InputMethodQuery {
         ^
~~~

~~~
In file included from /usr/local/opt/qt5/lib/QtWidgets.framework/Headers/QApplication:1:
In file included from /usr/local/opt/qt5/lib/QtWidgets.framework/Headers/qapplication.h:48:
/usr/local/opt/qt5/lib/QtGui.framework/Headers/qguiapplication.h:85:12: error: unknown type name 'QWindowList'
    static QWindowList allWindows();
           ^
/usr/local/opt/qt5/lib/QtGui.framework/Headers/qguiapplication.h:86:12: error: unknown type name 'QWindowList'
    static QWindowList topLevelWindows();
           ^
/usr/local/opt/qt5/lib/QtGui.framework/Headers/qguiapplication.h:138:12: error: unknown type name 'QFunctionPointer'
    static QFunctionPointer platformFunction(const QByteArray &function);
           ^
/usr/local/opt/qt5/lib/QtGui.framework/Headers/qguiapplication.h:143:16: error: no type named 'ApplicationState' in namespace 'Qt'
    static Qt::ApplicationState applicationState();
           ~~~~^
/usr/local/opt/qt5/lib/QtGui.framework/Headers/qguiapplication.h:164:38: error: no type named 'ApplicationState' in namespace 'Qt'
    void applicationStateChanged(Qt::ApplicationState state);
                                 ~~~~^
~~~


A little search on the web shows this is a problem of conflict with Qt4, and
that uninstalling Qt4 should help. Indeed, even just unlinking the `qt` formula
with [homebrew] makes the build pass:

~~~bash
brew unlink qt
~~~

Unfortunately, other programs are now failing:

~~~bash
$ qcachegrind 
dyld: Library not loaded: /usr/local/lib/QtGui.framework/Versions/4/QtGui
  Referenced from: /usr/local/bin/qcachegrind
  Reason: image not found
Trace/BPT trap: 5
~~~

So I had to find a better alternative!

[clang] — and probably all the compilers in their own way — has a flags that shows
the headers actually included during the parsing: `-H`.

In my case, for the Qt5 headers, it shows something like:

~~~
.. /usr/local/opt/qt5/lib/QtWidgets.framework/Headers/QApplication
... /usr/local/opt/qt5/lib/QtWidgets.framework/Headers/qapplication.h
.... /usr/local/include/QtCore/qcoreapplication.h
..... /usr/local/include/QtCore/qobject.h
~~~

The command line looks like that (I've suppressed everything not needed to make
it a bit easier to read):

~~~
c++ -isystem /usr/local/opt/qt5/lib/QtCore.framework/Headers \
  -iframework /usr/local/opt/qt5/lib \
  -isystem /usr/local/opt/qt5/lib/QtWidgets.framework/Headers \
  -isystem /usr/local/include \ 
  ...
~~~

The first two lines of includes are ok, but then, at the third line, it switched to Qt4
includes (note the lack of `qt5` in the path).

In `qapplication.h`, the header `QtCore/qcoreapplication.h` is included.
According to [Apple's documentation on frameworks], this means that the
`qcoreapplication.h` header could be located in the Qt5 framework
`QtCore.framework/Headers` (or `QtCore/PrivateHeaders`), if `QtCore.framework`
is in `/System/Library/Frameworks` or `/Library/Frameworks`, or if the base
directory of `QtCore.framework` is passed to the compiler with the flag
`-framework` or `-iframework`.

`QtCore.framework` is in `/usr/local/opt/qt5/lib`, which is not one of the default
framework directories, but the flag `-iframework` is properly used:

~~~
-iframework /usr/local/opt/qt5/lib
~~~

so this is why it works when there is Qt5 alone.

However, when Qt4 is also installed, the compiler seems to first look at the normal
includes before looking at the frameworks, and so the first `QtCore/qcoreapplication.h`
found is the one from Qt4, which is in the include dir for other libraries: `/usr/local/include`.

Qt5 also has its own include directory that contain the `QtCore` directory (and others)
as Qt4 does, and including that directory before the Qt4 directory is enough to fix
the build.

With CMake unfortunately, it is not included when using the standard
`find_package(Qt5...)` commands. It has to be done by hand, with

~~~cmake
include_directories(BEFORE /usr/local/opt/qt5/include)
~~~

or if you are not the developer, you can set the `CMAKE_CXX_FLAGS` option to
`-I/usr/local/opt/qt5/include`:

~~~
cmake -DCMAKE_CXX_FLAGS:STRING=-I/usr/local/opt/qt5/include ..
~~~

[Apple's documentation on frameworks]: https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPFrameworks/Tasks/IncludingFrameworks.html#//apple_ref/doc/uid/20002257-97563
[CMake]: http://www.cmake.org
[homebrew]: http://brew.sh/
[clang]: http://clang.llvm.org/
