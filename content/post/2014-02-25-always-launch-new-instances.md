---
title: Always launch new instances
date: 2014-02-25T19:21:41+00:00
tags:
  - reversing
---
If you have Windows 8.1, try this: hit the Windows key, start typing &#8220;not&#8221; until you see autocomplete for Notepad and hit Enter. Now type something in Notepad and repeat the same thing. Notice that when you hit Enter to launch Notepad again, it will _reactivate_ the previous running instance of Notepad. If you have multiple instances of Notepad, it will reactivate an arbitrary instance. I don&#8217;t know who the hell thought this is a good idea. Same thing essentially happens with any desktop app (including the Command Prompt which is another thing that infuriates me).

On Windows 8 there used to be this undocumented registry setting (`DesktopAppsAlwaysLaunchNewInstance`) that no longer works for search results in 8.1. So we have a problem to fix. I won&#8217;t go much into details, but if you want this fixed, you can do this:

  1. Install Debugging tools for Windows (free download from MSDN)
  2. Create a directory on your computer to store symbols in (I use c:\localsymbols)
  3. Create a shortcut on your desktop to run this: `[path_to_debugging_tools]\ntsd.exe -pn explorer.exe -pv -y SRV*[path_to_local_symbols]*http://msdl.microsoft.com/download/symbols -c "eb Windows_UI_Search!SearchUI::Data::SwitchToApp b8 00 00 00 00 c3; q"` (replace the two paths to point wherever you need)
  4. Double click the shortcut and repeat the above experiment. You are welcome.
