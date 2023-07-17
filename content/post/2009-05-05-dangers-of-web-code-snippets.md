---
title: Dangers of web code snippets
date: 2009-05-05T21:51:22+00:00
tags:
  - programming

---
Every programmer knows these situations: you are in the middle of programming something when you realize you need to do a thing you never did before. Instead of opening library/programming language/toolkit documentation, you just fire up google, type in a few keywords and import the first snipped that google finds straight into your source code.

I&#8217;ve hit these situations many times. The problem is that usually these snippets are just plainly wrong. Often, you end up importing source code from a 12-year old blogger with 3 months of experience in programming. Not really what you want to do in production code.

I&#8217;ve hit this situation today, again. I needed my C# WinForms application to go fullscreen. [Google][1] immediately yielded some results. First result was [this][2], the second one [this][3].

The first one didn&#8217;t really do what I needed. And when I saw the ugly P/invokes in the second one, I decided to do it myself.

The solution was dead simple: set `FormBorderStyle` to `None`, `TopMost` to `true` and `WindowStyle` to `Maximized`. That&#8217;s it. A fullscreen form without P/invokes or other voodoo, that will even work with Mono.

So, be careful with random web code snippets and _always_ read comments for the article where those snippets appear. They are usually pretty good at telling you if this is a bad solution and pointing you in the right direction.

 [1]: http://www.google.com/search?q=c%23+fullscreen
 [2]: http://www.codeproject.com/KB/cs/fullscreenmode.aspx
 [3]: http://social.msdn.microsoft.com/Forums/en-US/winforms/thread/c19872ab-8f3f-435f-a201-a36dea62ba98