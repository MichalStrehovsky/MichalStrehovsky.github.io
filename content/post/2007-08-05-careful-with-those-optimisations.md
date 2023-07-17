---
title: Careful with those optimisations
date: 2007-08-05T15:55:35+00:00
tags:
  - reversing
---
I was looking over some older computer magazines today and found a promising series of articles on anticracking in a Slovak IT magazine called [InfoWare][1]. I didn&#8217;t really have much time to read the whole series, so I just peeked at the enclosed source codes.

One code snippet caught my attention in particular. The code went like this:

```c
float sumOfNumbers = 90 - 1 + 0.03 + 50 + 300 + 0.3 - 50;
if (sumOfNumbers == 389.33) {
    // everything's all right
} else {
    // now confuse the cracker
}
```

The text around this snippet was talking something about making constants in programs more confusing.

Well &#8211; in source code, this is really terrifying and confusing. It works well, if you want to protect your _source code_ against modifications by the mystified programmers (or by you). It will hardly confuse a cracker, who sees only the binary version. The reason? Compiler optimizations.

I made a little experiment with this code:

```c
float f = 3.1 + 5.8 + 1.1;

if (f == 10)
    printf("Aloha");
```

I compiled it in Visual C++ with full compiler optimizations (the &#8220;release mode&#8221;) and glanced at the produced code. The above code was compiled into this:

```asm
push Test.004020E4                       ; /format = "Aloha"
call dword ptr ds:[&lt;&#038;MSVCR80.printf>]    ; \printf
```

Only the printf call was left from the original code. Not very confusing anymore, is it?

 [1]: http://www.infoware.sk