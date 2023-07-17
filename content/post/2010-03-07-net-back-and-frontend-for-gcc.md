---
title: '.Net back- and frontend for GCC'
date: 2010-03-07T16:30:01+00:00
tags:
  - dotnet
---
I recently stumbled upon a very interesting project &#8211; a CIL (.Net assembler) backend and frontend for GCC.

Suppose you have source code in one of the languages supported by GCC (C, C++, Ada, Fortran,&#8230;) and you want to use the code from managed code (say, a C# application). A CIL backend would allow you to do just that &#8211; compile and link the C code into a .Net assembly so you can link it with C# code.

Now suppose you have a managed .Net assembly and want to compile it to native code for one of the architectures supported by GCC (ARM, amd64, x86, MIPS,&#8230; on Linux, Windows, Darwin,&#8230;). A GCC frontend allows you to do just that. Take .Net CIL code and compile it to native code.

# GCC-CLI

There is a branch in GCC allowing you to do just that. The branch is called [GCC-CLI][1]. Even though it&#8217;s still very experimental, it already allows you to do a few interesting things. For example: compile C code into managed code using the CIL backend and then use the CIL frontend to compile the managed code to native code.

The backend is quite useful already. I was able to compile my [pet C compiler][2] into a fully managed EXE. Not even the Managed C++ compiler from Microsoft can do that! (Managed C++ creates mixed-mode assemblies, which are not fully managed and even if you try to get past this limitation with compiler switches, you get burned by the CRT library.)

The frontend only supports a limited subset of CIL and lacks a runtime library that would do garbage collection and other services (like GCC does with Java &#8211; libgcj). So it&#8217;s not that useful, yet. But the potential is huge!

# A Windows binary specially for the readers of my blog :)

You can try it out yourself. The build instructions are [here][3]. You can build it with MSYS and MingW on Windows, but the build process is quite painful. If you don&#8217;t want to build it yourself, you can download the Windows binary of GCC with 32bit CIL backend [here][4] (7-zip archive &#8211; I hope you know what to do with that). Download the archive even if you want to build it yourself. There is a text document with a bunch of notes on how I built it.

Unpack the archive somewhere and run bin\_BUILDTEST.cmd to build a hello world C application. To run the generated _test.exe, you&#8217;ll need the assemblies from lib\ to be present in GAC, or in the same folder as the EXE (just copy them over to bin\ and be done with it). Otherwise it will crash. Also, .Net framework is necessary (obviously).

Only C is supported in my build. I was not successful with C++ (it looks like the backend can&#8217;t handle exceptions) and didn&#8217;t try anything else.

 [1]: http://gcc.gnu.org/projects/cli.html
 [2]: http://migeel.sk/programming/mscc/
 [3]: http://gcc.gnu.org/viewcvs/branches/st/README?view=markup
 [4]: http://downloads.migeel.sk/gcc-cli_win32_svn-5.3.2010.7z