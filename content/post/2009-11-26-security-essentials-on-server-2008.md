---
title: Security Essentials on Server 2008
date: 2009-11-25T22:12:48+00:00
tags:
  - reversing
---
**Update 15. 3. 2010:** this technique will fail at the genuity verification step of setup with recent Security Essentials installers. So it doesn&#8217;t work anymore. (And I will not be writing an update of this post, because I have moral problems with bypassing genuity verifications, sorry.)

I was getting a lot of e-mail about my recent article on installing [Microsoft Security Essentials on Windows Server][1]. Because the article shows up pretty high on google when searching for &#8220;security essentials windows server&#8221;, people kept asking me for a step-by-step tutorial. Originally, I didn&#8217;t want to do it because if someone isn&#8217;t able to follow the brief instructions I wrote in the original article, he won&#8217;t be able to fix problems that might show up after installing updates to Security Essentials. But whatever. Fasten your seat belts because here it comes:

REMEMBER: THIS IS NOT SUPPORTED BY MICROSOFT. IF YOU FOLLOW THESE INSTRUCTIONS, YOUR COMPUTER MIGHT BECOME UNBOOTABLE, SECURITY ESSENTIALS MIGHT APPEAR THEY WORK BUT THEY SILENTLY WON&#8217;T. ALSO, NIGHT ELVES WILL EAT ALL GROCERIES FROM YOUR FRIDGE WHILE YOU SLEEP.

This will only work for 64bit versions of Windows Server.

  1. Follow my [original article][1] up to (and including) the point where you copy the unpacked setup files.
  2. Download and install [Microsoft Debugging Tools for Windows][2]. Choose the version of debugging tools matching your operating system version.
  3. Find WinDbg in the start menu and launch it elevated.
  4. In the WinDbg menu choose File -> Open Executable and open the unpacked Setup.exe. Say no to the question about saving workspace.
  5. A new window will show up. There will be a box for entering commands at the bottom of the new window. This is the command line.
  6. Write this into the command line and hit enter: `bp ntdll!RtlGetNtProductType "as /x ReturnValue rcx; gu; ed ReturnValue 1; g"`. This will set up a breakpoint that will modify the return value of RtlGetNtProductType (anyone has a clean way of doing this for 32bit Windows?)
  7. Write `g` on the command line and hit enter to resume the installer.
  8. The installer will start &#8211; focus it&#8217;s window and click Next.
  9. Go back to WinDbg. Hit Ctrl-Break. Type the command `bc *` to remove the breakpoint and after that `g` to resume the setup. Finish installing. Done.

If at some point in the future something breaks, don&#8217;t go back crying to me. I warned you.

 [1]: http://migeel.sk/blog/2009/10/17/security-essentials-on-windows-server/
 [2]: http://www.microsoft.com/whdc/DevTools/Debugging/default.mspx