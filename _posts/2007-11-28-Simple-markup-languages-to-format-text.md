---
title: Simple markup languages to format text
---

One of the most boring task I know is to format a text in *modern word processor* — read: a program where you can change the police of some text with a graphical user interface. Some of those programs are [Microsoft Word][1], [OpenOffice Writer][2], or [Journler][3]. The main goal of this last program is not to be a word processor, and it is very well made to organize a bit the daily notes, but it is one of the most painful program I know to format some text.

The main drawback, if we exclude the poorly made user interface (the journler case), is the lack of global way to format things — make all the same elements look the same. Writer and Word are providing some style support, but it's still very easy to *not* use them, and, thanks to the [WYSIAYG][7] paradigm, not even know it has not been used until someone try to change the style. Any collaborative work on such documents quickly become a real nightmare.

The only nice alternative I know is the formating with a markup language: the text is combined with some elements (of text) dedicated to the formating (or to more descriptive things). The marked text is then processed to produce a nice formatted text in the user preferred format.

The markup syntax can be extremely simple, and made to be human readable, as in [markdown][4] (used on this blog) or [asciidoc][5], or much complex as in [Latex][6].
Because those methods are fully based on text, they are very easily usable in a collaborative work with the help of any [version control system][8].
With all of them, there is never anything hidden. With the simplest of them, writing a nice formated text is as simple as writing an email, with a way nicer visual result.

[1]: http://office.microsoft.com/en-us/word/FX100487981033.aspx
[2]: http://www.openoffice.org/product/writer.html
[3]: http://journler.com/
[4]: http://daringfireball.net/projects/markdown/
[5]: http://www.methods.co.nz/asciidoc/index.html
[6]: http://www.latex-project.org/
[7]: http://en.wikipedia.org/wiki/WYSIAYG
[8]: http://en.wikipedia.org/wiki/Version_control_system
