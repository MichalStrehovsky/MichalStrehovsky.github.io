---
title: "Top 3 whole program optimizations in .NET 8 native AOT"
slug: "top-3-whole-program-optimizations-for-aot-in-net-8"
date: 2023-11-22T00:45:19+09:00
image: title.jpg
description: A look at some of the whole program optimizations that are done when you compile your app as native AOT.
tags:
  - dotnet
---

When you compile your .NET program as [Native AOT](https://learn.microsoft.com/dotnet/core/deploying/native-aot/), the whole program and its runtime data structures get compiled to native representation. There's no virtual machine to offer various services at runtime. This constrains what the program can do and requires giving up certain conveniences of .NET, such as the ability to load new types and assemblies at runtime (i.e. Reflection.Emit and support for managed plug-ins). But while you lose some things, you gain other things: faster startup, less memory use, smaller size, and the ability to optimize your app using whole program view.

Normally, optimizations in compilers are done on a per-method basis: the compiler looks at what the method does and figures out a way to do that most efficiently using the information seen in the method body and input from the type system. A whole program optimization looks at the entire program, and feeds that information into compilation of individual methods on top of the existing data from the type system and the method body itself. This allows generating better code.

Whole program view depends on knowing what the entire program does at the time the method is compiled: before the compiler starts generating code for the first method, it needs to know what all the methods in the program do. AOT compilers are well positioned to have this knowledge. The constraints that AOT imposes (no ability to generate or load new code at runtime) play well with this.

Here are the top optimizations done by native AOT in .NET 8 using whole program view.

# 1. Sealing classes that are not inherited by any other class

```csharp
using System.Runtime.CompilerServices;

CallToString(new Base());

// NoInlining so that codegen doesn't inline it into callsite
[MethodImpl(MethodImplOptions.NoInlining)]
static string CallToString(Base b) => b.ToString();

class Base
{
    public override string ToString() => "Base";
}

class Derived : Base
{
    public override string ToString() => "Derived";
}
```

In the above program, the `CallToString` method gets compiled into following CPU instructions:

```asm
// Dereference `this` pointer
cmp      byte  ptr [rcx], cl

// Load address of a "frozen string literal", a System.String instance
// that is allocated in the data segment of the executable (outside GC heap).
lea      rax, gword ptr [(reloc 0x422978)]

// Return
ret
```

Observations:
* `Base.ToString` was inlined into `CallToString`, despite this being a virtual call. The generated code performs a null check in the first instruction (to trigger a `NullReferenceException` if the parameter was null), loads a string literal “Base” and returns. 
* This is possible because the whole program view realized that there's no other class that could be typed as `Base` than `Base` itself.
* The program has a `Derived` class in it that derives from `Base` and overrides `ToString`, but it got optimized away because it's unused.
* If loading new code was supported, this optimization would be illegal because at any point `Derived` could be loaded or a new class deriving from `Base` could be created with `Reflection.Emit`, or `Assembly.LoadFrom`. We'd either need an extra if check to check the type is indeed `Base` or we'd need a way to invalidate all methods that had wrong assumptions at the point when the assumption becomes invalid.
* At some point in the future, we might also be able to optimize away the null check at the beginning of `CallToString` because the whole program view can see this method is never called with a null parameter (the whole program view also knows which methods are called indirectly through delegates or reflection – and this method isn't).

# 2. Devirtualizing interface calls to interfaces with few implementors

```csharp
using System.Runtime.CompilerServices;

CallFrob(new Fooer1(), 1, 1);
CallFrob(new Fooer2(), 1, 1);

[MethodImpl(MethodImplOptions.NoInlining)]
static int CallFrob(IFoo foo, int x, int y) => foo.Frob(x, y);

interface IFoo
{
    int Frob(int x, int y);
}

class Fooer1 : IFoo
{
    public int Frob(int x, int y) => x + y;
}

class Fooer2 : IFoo
{
    public int Frob(int x, int y) => x - y;
}
```

In the above program, `CallFrob` gets compiled into following instructions:

```asm
  // Load MethodTable of Fooer1
  lea      rax, [(reloc 0x420508)]

  // Compare type of first parameter with Fooer1
  cmp      qword ptr [rcx], rax

  // If not equal, jump to IG04
  jne      SHORT G_M000_IG04

G_M000_IG03:
  // Fancy way to add rdx and r8 and put result in eax
  lea      eax, [rdx+r8]

  // Jump to end of method
  jmp      SHORT G_M000_IG05

G_M000_IG04:

  // Move second parameter into eax
  mov      eax, edx

  // Subtract third parameter from eax
  sub      eax, r8d

G_M000_IG05:
  // Return
  ret
```

Observations:
* The code generated for `CallFrob` is essentially `foo.GetType() == typeof(Fooer1) ? x + y : x – y`
* The compilation process figured out there are only two possible types that could be `IFoo` at runtime. If it's not `Fooer1` (or null, which would have thrown when retrieving type identity), it must be `Fooer2`.
* Both implementations were small enough to inline so they got inlined.
* If loading new code was supported (JIT case), a fallback option would need to be generated, or we'd need a way to invalidate this code if a new implementation of `IFoo` is loaded.

# 3. Interpreting static constructors at compile time and inlining readonly information

```csharp
using System.Runtime.CompilerServices;

class Primes
{
    readonly static int ThePrime;

    [MethodImpl(MethodImplOptions.NoInlining)]
    static int Main()
    {
        return ThePrime;
    }

    static Primes()
    {
        const int Seq = 6;

        // Computes the Seq-th prime number and stores it in ThePrime.

        var candidates = new bool[(Seq + 1) * (Seq + 1)];
        for (int i = 2; i < candidates.Length; i++)
        {
            for (int j = i * 2; j < candidates.Length; j += i)
                candidates[j] = true;
        }

        int numPrime = 0;
        for (int i = 2; i < candidates.Length; i++)
        {
            if (!candidates[i])
                numPrime++;

            if (numPrime == Seq)
            {
                ThePrime = i;
                break;
            }
        }
    }
}
```

The code generated for `Main` is:

```asm
mov      eax, 13
ret
```

Observations:
* The method body of Main simply returns a constant 13 (the sixth smallest prime).
* The static constructor got interpreted at compile time and when code generation saw the readonly bit on the field, it simply asked for the constant value to inline into code.
* The native code for the static constructor was not even generated.
* In .NET 9, this optimization gets extended to also work without `readonly` because whole program optimization can see this field is never assigned outside the static constructor (and there's no reflection access to it either). This can be useful even when it's not possible to mark something `readonly` in source code (because it's modified), but the code doing the modification got optimized away and the field became _effectively read only_.

# Summary

Being able to make "closed world" assumptions about the program opens the door to optimizations that are not possible in an open world. Native AOT first debuted in .NET 7 and there's still a long road towards enabling more of these "closed world" optimizations.

Of course JIT-compiled .NET also has tricks up it's sleeve, such as ability to monitor program behavior at runtime and recompile methods based on observed runtime behaviors (dynamic PGO). You'll often see JIT-based .NET beat AOT-based .NET in peak throughput, however these closed world optimizations can sometimes give edge to the AOT-compiled one.
