---
title: Compiling GDB under Windows
date: 2009-04-15T21:10:42+00:00
tags:
  - misc
---
Just a quick post to make things simpler for everyone who is googling for help when compiling GDB debugger under Windows.

There are two small oddities that require edits in the source code when compiling GDB 6.8 using [MingW][1] with MSYS.

  * In sim\common\sim-signal.c: replace `#ifdef _MSC_VER` with `#ifdef _WIN32`
  * In gdb\tui\tui-io.c: find the line with `/* #undef TUI_USE_PIPE_FOR_READLINE */` and uncomment it

After you change this, just &#8220;configure&#8221; and &#8220;make&#8221; and you are good to go.

And if you want the TUI (textmode GUI interface), install [PDCurses][2] to your MSYS directory (and create a link from MSYS\lib\libpdcurses.a to MSYS\lib\libcurses.a; not sure it&#8217;s neccessary, but that&#8217;s what I did) before running the configure script.

 [1]: http://www.mingw.org/
 [2]: http://sourceforge.net/project/showfiles.php?group_id=2435&package_id=14876