---
title: GDB and QEMU on Windows
date: 2009-04-21T13:01:34+00:00
tags:
  - tricks

---
A few weeks ago I started to work on a small operating system for a MIPS-based development motherboard. When thinking about a development toolchain, I immediately looked at one of my favorite emulators &#8211; [QEMU][1].

QEMU has a few nice features that make development of operating systems easier than ever. One of these features is the `-kernel` command line parameter that loads a custom operating system kernel right into memory without the need to write a custom boot loader. Another useful command line option is `-s` which starts a [GDB][2] server inside QEMU so you can connect to it with GDB (with the command `target remote :port_number`) and debug your loaded kernel with full symbols.

At first, this didn&#8217;t work for me. GDB refused to connect to the server for no apparent reason (_No connection could be made because the target machine actively refused it._). It took me almost half an hour to figure out where the problem was: QEMU was opening an IPv6-only port and GDB was using IPv4. Quick fix: open up gdbstub.c in QEMU sources, locate the line where the connection string is being created (in QEMU 0.10.2 it&#8217;s the line 2300; there is a string that says: `tcp::%d,nowait,nodelay,server`) and fix the connection string to look like this: `tcp::%d,nowait,nodelay,server,ipv4`.

 [1]: http://www.nongnu.org/qemu/
 [2]: http://www.gnu.org/software/gdb/