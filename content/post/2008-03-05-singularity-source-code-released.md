---
title: Singularity source code released
date: 2008-03-05T21:51:47+00:00
tags:
  - dotnet
---
Microsoft has finally made the source code of it&#8217;s research OS called &#8220;[Singularity][1]&#8221; available to general public.

Singularity is a prototype operating system coded almost entirely in managed code. It&#8217;s written using Sing#, a language derived from Spec#, which itself has roots in C#. Spec# adds Eiffel-like contracts (loop invariants, preconditions, postconditions, etc.) to C#. Sing# extends Spec# with low-level constructs required for operating system development and channels required for communication within Singularity&#8217;s microkernel.

Okay, now what does this mean?

  * Singularity&#8217;s code can be mechanically proved correct. This can easily reduce number of possible programming errors by orders of magnitude.
  * Singularity&#8217;s strong typing creates impenetrable memory boundaries within operating system components and processes. This allows execution of _everything_, including user processes in ring 0. No more CPU cycles wasted by context switching.
  * And much much more :)

Other projects attempting to create a CLI-based operating systems are [SharpOS][2] (which unfortunatelly uses the aggressive GPLv3 license) and [Cosmos][3] (released under a BSD license).

EDIT: I almost forgot the download link for Singularity; you can get it from [Codeplex][4].

 [1]: http://research.microsoft.com/os/singularity/
 [2]: http://www.sharpos.org/
 [3]: http://gocosmos.org/
 [4]: http://www.codeplex.com/singularity