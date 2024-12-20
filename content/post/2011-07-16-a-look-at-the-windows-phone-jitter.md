---
title: A look at the Windows Phone JIT compiler
date: 2011-07-16T06:07:04+00:00
tags:
  - dotnet
  - reversing
---
When optimizing a very hot path in my code, I sometimes find it useful to see what code the compiler is generating for me. Many times I can spot things that can be easily fixed by rearranging code or adding some typecasts.

But getting my hands on the CLR JIT-generated code disassembly on the Windows Phone was not easy. If you think it&#8217;s as easy as breaking into the Visual Studio debugger and pressing Ctrl-Alt-D, you&#8217;ll be disappointed:

```
No disassembly available.
```

Luckily for us, at least the Memory window in Visual Studio still works. Getting our hands on the JITted code will be hard, but not impossible.

Let&#8217;s write a method that will be easy to spot in the memory window:

```csharp
public partial class MainPage : PhoneApplicationPage
{
  // ...
  public uint Foo()
  {
    return 0xDEADBEEF;
  }
}
```

Now add a call to this method in PhoneApplicationPage_Loaded and set up a breakpoint after the method call to make sure it&#8217;s JITted when the breakpoint is hit. Deploy your project to the emulator and break into the debugger. Now let&#8217;s find the method in memory.

Because we can&#8217;t use unsafe code on Windows Phone, and the System.Runtime.InteropServices.Marshal class is off limits, we have to turn our hopes to reflection. Luckily for us, the System.Reflection.MethodInfo class contains a field named MethodHandle whose Value points to some kind of internal CLR runtime structure (MethodDesc?). Even though it&#8217;s undocumented, we can probably recognize pointers in it and try our luck disassembling memory they point to.

Open the Immediate window in Visual Studio and type:

```
?typeof(MainPage).GetMethod("Foo").MethodHandle.Value
```

Executing the above statement in my debugging session gave me `0x0658c910`. Looking at that offset in the memory window gave me this:

```
0x0658C910  e8 a0 b6 03 b8 c5 58 06 0e 00 00 06 01 00 86 00
```

Following the first pointer to `0x03b6a0e8` (remember, little-endian) will give you this:

```
0x03B6A0E8  5a 89 55 08 83 c4 d0 89 2c 24 8b ec b9 e8 a0 b6  
0x03B6A0F8  03 33 c0 89 45 14 89 4d 0c 89 6e 14 89 45 2c 05  
0x03B6A108  00 00 00 00 05 00 00 00 00 ba ef be ad de 89 55  
0x03B6A118  2c 05 00 00 00 00 05 00 00 00 00 8b 55 2c 8b 6d  
```

See the string `ef be ad de`? That has to be our code! Dump the contents of the memory window to a file and save it.

Now fire up your favorite ARM disassembler and load the dumped bytes at offset `0x03B6A0E8`. Does it look like trash? It is trash! That&#8217;s because the code you are actually looking at is x86, not ARM. How is that possible? The JIT compiler in the Windows Phone emulator produces x86 code. It actually makes sense, because running native code is faster than emulating ARM code. This is probably the reason why the Phone emulator needs hardware virtualization and can&#8217;t run under Hyper-V. Most of it runs as i386 code! To see the actual ARM code of the method, you have to dump it from your physical device.

But because we already have the x86 code dumped, let&#8217;s have a look at it:

```asm
loc_3B6A0E8:                            ; DATA XREF: seg000:03B6A0F4
                pop     edx
                mov     [ebp+8], edx
                add     esp, 0FFFFFFD0h
                mov     [esp], ebp
                mov     ebp, esp
                mov     ecx, offset loc_3B6A0E8
                xor     eax, eax
                mov     [ebp+14h], eax
                mov     [ebp+0Ch], ecx
                mov     [esi+14h], ebp
                mov     [ebp+2Ch], eax
                add     eax, 0
                add     eax, 0
                mov     edx, 0DEADBEEFh
                mov     [ebp+2Ch], edx
                add     eax, 0
                add     eax, 0
                mov     edx, [ebp+2Ch]
                mov     ebp, [ebp+0]
                add     esp, 34h
                mov     [esi+14h], ebp
                jmp     dword ptr [ebp+8]
```

You&#8217;ll probably quickly notice 4 things:

1. It&#8217;s a particularly chatty way of saying `mov edx, 0xDEADBEEF; ret`.
2. The method has a weird prolog and epilog.
3. The method doesn&#8217;t follow the Intel ABI.
4. The method uses a rather big `add eax, 0` instruction as a `nop`. A `nop` with side effects.

First point can partially be explained by the fact that I was running an unoptimized version (the Debug project configuration), but is closely related to the second, third and fourth point: what we are looking at is code generated by an ARM code generator that was hacked to generate x86 code! The last instruction in the listing is a dead giveaway.
    
Now let&#8217;s look at the code dumped from my actual device (disassembled with standard 32bit ARM instruction encoding):
    
```asm
                ADD     R9, SP, #0
                STR     PC, [R9,#0xC]
                MOV     R2, #0
                STR     R2, [R9,#0x14]
                STR     R9, [R11,#0x14]
                STR     R2, [R9,#0x2C]
                ANDEQ   R0, R0, R0
                ANDEQ   R0, R0, R0
                LDR     R1, =0xDEADBEEF
                STR     R1, [R9,#0x2C]
                ANDEQ   R0, R0, R0
                ANDEQ   R0, R0, R0
                LDR     R1, [R9,#0x2C]
                LDR     R9, [R9]
                ADD     SP, SP, #0x34
                STR     R9, [R11,#0x14]
                LDR     LR, [R9,#8]
                BX      LR
```
    
This code feels much more natural than its x86 version. Now let&#8217;s look at how the code looks if we enable optimizations (the Release configuration):


```asm
                STR     LR, [R9,#8]
                SUB     SP, SP, #0x2C
                STR     R9, [SP]
                ADD     R9, SP, #0
                STR     PC, [R9,#0xC]
                MOV     R2, #0
                STR     R2, [R9,#0x14]
                STR     R9, [R11,#0x14]
                ANDEQ   R0, R0, R0
                LDR     R1, =0xDEADBEEF
                LDR     R9, [R9]
                ADD     SP, SP, #0x30
                STR     R9, [R11,#0x14]
                LDR     LR, [R9,#8]
                BX      LR
```

You&#8217;ll probably notice the code is still not very optimal. As it turns out, the JIT code generator heavily favors code generation speed against code quality. To get most of your CPU cycles, you have to be very careful about how you write your code.
    
I hope this short post will be useful to you when doing your own Windows Phone .NET code generation investigations. I plan to follow up with some notes on what optimizations you can expect from the Windows Phone CLR code generator that I gathered while optimizing [my GameBoy emulator][1] to run on my phone.
    
Two more useful thing to note: when dumping the method to a file, look at the bytes preceding the method body. Every method has some kind of a header that has (apart from other stuff) 2 pointers in it: pointer to the end of the method body and a pointer to the end of method body including the literal pool. It seems like the header is different depending on whether you deploy a retail or debug configuration.
    
Many times it&#8217;s easier to just dump the whole JIT code heap instead of doing it method by method. After you find the address of your method, just scroll up in the Memory window until you hit uncommited memory region (filled with question marks). Then dump everything starting from there to the end of the heap (a big block of zeroes or question marks).
    
When it comes to choosing a disassembler, you can try GNU objdump, but if you want something painless, IDA Pro is probably your only option. Get the demo and use [this workaround][2] to open raw binaries in it.

 [1]: http://migeel.sk/projects/mgbemu/ "MGBEmu"
 [2]: http://www.freemyipod.org/wiki/Working_with_binaries