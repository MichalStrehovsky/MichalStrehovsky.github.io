---
title: A few useful registry hacks
date: 2013-04-20T23:37:34+00:00
tags:
  - tricks
---
Here are a few registry hacks I find useful. I&#8217;m posting this mostly for my own reference when setting up new Windows 8 + Visual Studio 2012 machines.

**Always launch new instances of desktop apps**  
There is nothing more annoying than when I try to open a new command prompt or Notepad and the immersive launcher (&#8220;start screen&#8221;) decides to activate my existing instance. Thank you launcher, that&#8217;s exactly what I wanted. Not.  
Create a DWORD value at _HKEY\_CURRENT\_USER\Software\Microsoft\Windows\CurrentVersion\ImmersiveShell\Launcher_ named DesktopAppsAlwaysLaunchNewInstance with value 1 and Launcher will never try to be &#8220;helpful&#8221; again.

**GET RID OF ALL CAPS MENUS IN VISUAL STUDIO**  
Do your eyes a favor and set the DWORD at _HKEY\_CURRENT\_USER\Software\Microsoft\VisualStudio\11.0\General_ named SuppressUppercaseConversion to 1.

**Prevent corners from capturing the mouse on multimon setups**  
With multiple monitors, more often than not my mouse gets captured in the corners because shell thinks I would like to use the charms bar. I never use the charms bar on my desktop, hence this is more annoying than useful. Thanks, but no thanks. Setting the string at _HKEY\_CURRENT\_USER\Control Panel\Desktop_ named MouseCornerClipLength to 0 fixes this.

And I have one more non-registry efficiency improvement tip. Since the metro apps for Music/Videos/Pictures are pretty much useless on desktop but Windows defaults to using them for all music/video/picture formats, uninstall them. You will save yourself the trouble of changing the associations for every single file format supported by them and having a perfectly fine desktop alternative.