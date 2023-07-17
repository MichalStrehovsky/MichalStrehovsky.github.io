---
title: Advanced self-modifying code
date: 2007-08-02T07:46:18+00:00
tags:
  - reversing

---
Self-modifying code (SMC) belongs to the strongest weapons of software protection programmers. I already presented the basic principles behind SMC in my [series of cracking prevention articles][1]. In this article we are going to dive deeper and take a closer look at some advanced techniques like polymorphism and metamorphism.

Polymorphism first appeared in a computer virus called 1260 as a method designed to hide the virus from anti-virus software. The anti-virus software at that time used patterns to identify malicious code inside of executables. Because polymorphism made the representation of the virus code different in each infected file, the anti-virus creators could not find a unique pattern identifying it.

The concept was simple: at the beginning of the virus, there was a simple decryption routine that decrypted the rest of the virus in memory. Then the actual virus code was run. When the virus found its new victim, it changed the decryption key a little, encrypted itself with this new key and placed its encrypted code together with decryption code into the executable file.

# Polymorphism: the easy way

The easiest approach to polymorphism looks like this:

```asm
mov al, 12h ; set the key
mov edi, codeEnd ; starting address
mov ecx, codeEnd - codeStart ; length of encrypted block

; now decrypt the code, starting from the last byte
decryptLoop:
xor byte [edi], al ; decrypt byte
dec edi ; move to the next byte
loop decryptLoop

codeStart:
; put encrypted code here
codeEnd:
```

This kind of polymorphism is easy to implement and seems as a pretty good weapon. But let&#8217;s have a look at this code from a crackers point of view: the code between codeStart and codeEnd is encrypted, so he can&#8217;t see what will happen after the loop instruction. The easiest method to go through this problem is to place a hardware or memory breakpoint after the loop instruction and wait for the code to decrypt (BPX breakpoint won&#8217;t do here because it would alter the byte after the loop instruction &#8211; this would result in garbage after the decryption).

The point of polymorphism is to force the cracker to run our program &#8211; to make static disassembly useless. The problem with this polymorphic engine is that it&#8217;s too transparent &#8211; the cracker doesn&#8217;t have to run this code. He just has to look at the decryption routine and write a macro for IDA (or similar disassembler) to decrypt the protected code for him. Then he can NOP the decryption code out of the executable.

# Advanced polymorphism

More advanced polymorphic engines not only change the key protecting the sensitive code &#8211; they also change the algorithm doing the decryption. A good polymorhic engine usually has these features:

  * generates different instructions which do the same thing
  * swaps groups of instructions
  * creates calls to dummy routines
  * generates lots of conditionals jumps
  * embeds anti-debugging tricks
  * inserts junk instructions into real code

A combination of these techniques makes debugging of the decryption routine really hard and painful.

# Generating different instruction which do the same thing

As a programmer you already know that there is always more than one way to do one thing. As an example, let&#8217;s solve this task: set the EAX register to value 100h.

```asm
; simple assignment
mov eax, 100h

; using stack
push 100h
pop eax

; first zero register, then do a binary or
sub eax, eax
or eax, 100h

; this will even hide the assigned value from cracker's eyes
mov eax, 12345778h ; 12345778h = 100h xor 12345678h
xor eax, 12345678h
```

# Intermediate languages

Implementation of this is pretty straightforward. To represent the polymorphic decryptor, we create an intermediate language (a language of an abstract machine) represented by triplets. The internal representation of the decryptor will look like this:

```
[move, eax, 100h]
[jump, label_id, null]
[increment, eax, null]
```

Code generator will then go through the intermediate code and generate native code for each triplet. Each triplet will have one or more native code alternatives and the code generator will always pick one at random.

The representation using an intermediate language can also assist us in another tasks like swapping instructions or inserting garbage code into real code. Theory behind intermediate languages and optimizations (i.e. modifications of existing code while preserving the functionality) is explained in every book about compiler design.

# Problems with classical polymorphism

The biggest problem with classical polymorphism is that after the polymorphic decryption routine finishes, the sensitive code is left naked in memory. This means that if the cracker manages to get thought all the weird and hard-to-debug computer-generated code, he will find clean and comprehensible original code. This is something we need to prevent.

To avoid this, we can divide our code into smaller modules and put each of them into its own polymorphic envelope. This will make crackers life harder, because he will never see whole code at once and he will have to trace through the polymorphic decryptors annoyingly often.

But even this approach has its downside: it is the fact that the cracker is still given the opportunity to see the comprehensible original code.

# Metamorphism as an effective weapon

The solution of this problem is called metamorphism. From the outside it is similar to polymorphism &#8211; it creates different code for each application. But the key difference between polymorphism and metamorphism is that while polymorphism encrypts the sensitive code and creates a unique decryptor for it, metamorphism morphs the sensitive code to make it almost impossible to understand by a human.

With metamorphism it&#8217;s possible to create kilobytes of morphed code from several bytes of original code. Manual tracing of such code can easily take days or even weeks of hard work, with poor results (the cracker will never see the original code as with polymorphism).

# Metamorphism &#8211; an example

Metamorphic engine first takes existing code, analyzes it using an internal disassembler, morphs the internal representation of code and then generates morphed native code. Let&#8217;s have an example:

```asm
mov eax, 1h
mov ecx, Ah
```

The resulting morphed code can look like this:

```asm
xor eax, eax
inc eax
sub ecx, ecx
inc ecx
sal ecx, 2
inc ecx
sal ecx, 1
```

# Implementation details

The main difference between implementation of polymorphism and metamorphism lays in the fact that polymorphism doesn&#8217;t change the original code. It only hides it.

On the other hand, metamorphism changes the original code and thus has to cope with several problems:

  * Code flow: because each instruction is replaced with several new instructions, the length of code blocks changes. Engine has to detect and repair all jumping coordinates or function calls within the code to match new positions of code blocks.
  * Registers used as pointers: the same problem as with code flow.
  * Detecting data in code: most compilers today place some data in the text section of executable, together with code (e.g. between functions). An attempt to handle data as code (i.e. mutate it) could have fatal consequences.

This is the reason why metamorphism is never used for whole application, only for the protection itself.

# Partial and full metamorphism

Because of great complexity of the task of writing a metamorphic engine, many commercially available protections resort to partial metamorphism. They decide not to write a full morpher but select only a small subset of instructions that will be morphed. The other instructions are left without change.

This approach fulfills the goal of metamorphism only partially. While it&#8217;s still harder to understand the generated code, it&#8217;s not as hard as with full metamorphism. The reason for this is that the subset of affected instructions is usually too small to generate sufficient amount of confusing code.

The complexity of writing a full metamorphic engine is also proven by the fact, that at the time of writing this article, the only commercially available protection offering full metamorphism was SVKP 2.0. You can find it at [www.defendion.com][2].

 [1]: http://migeel.sk/articles
 [2]: http://www.defendion.com