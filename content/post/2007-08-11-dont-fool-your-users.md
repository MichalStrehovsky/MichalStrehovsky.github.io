---
title: Don't fool your users
date: 2007-08-11T11:27:31+00:00
tags:
  - programming
---
Today, I finally got into reading the series on anticracking I mentioned in my [previous post][1]. In one of the articles, I found this suggestion on what to do if you detect that your program is being crackedâ€ :

> Instead of crashing the program, you should wait several days and then change the way the program reacts. For example, in a graphical program when the user of illegal version picks green colour, the program will draw with blue colour. 

The intention of this is clear: discrediting the cracker. If he doesn&#8217;t notice this additional protection layer and ships his (unfinished) crack, his credit among the cracker community will be degraded.

In reality, though, the one with degraded credit will be you. _&#8220;Do not use the X program. It&#8217;s full of bugs and works in an unpredictable way.&#8221;_

People do not usually associate bugs in programs with unfinished cracks. If a program works just fine after it was cracked, people tend to forget that the program was ever cracked. All errors that show up after a certain period of time will be automatically associated with you.

If you want to include delayed checks in your protection, make sure they behave in a direct way. You detected that your program is partially cracked? Show a message boxâ€¡. Display a message on the application title bar. Inform your users. Do not let them make false assumptions about your program.

There are many ways how to detect this &#8211; a checksum doesn&#8217;t match, the registration verification procedure returned `true` even though the code supplyed was intentionally not valid, etc.

Of course, do not forget to hide the message in the code appropriately. You don&#8217;t want to bring the attention to this code, do you?

 [1]: http://migeel.sk/blog/2007/08/05/careful-with-those-optimisations/