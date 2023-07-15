---
title: "Building a universal Windows 7/Windows 10 .NET EXE"
date: 2021-04-19T15:35:12+09:00
description: Build a .NET 3.5 app that will run on Windows 10 without installing .NET 3.5.
---

The problem with building a .NET (classic) executable that runs on both clean Windows 7 install and on Windows 10 is that Windows 7 only ships with .NET 3.5 inbox and Windows 10 ships with .NET 4.X. A .NET 3.5 executable will not run on a (clean install) Windows 10 directly. It can be coerced to do so in multiple ways, but none of them are "worry-free single file" solutions (config file, registry settings, environment variables, etc.).

One of the solutions is to set `COMPLUS_OnlyUseLatestCLR` environment variable to `1` before the process starts. This will allow .NET 4.X to take over execution of the program. This still doesn't qualify as "worry-free" because we need a batch file or something else to set the envionment for us before the process start (it's too late once `Main` is executing).

# One weird trick to run the same executable on both Windows 7 and Windows 10

When I said we need to set `COMPLUS_OnlyUseLatestCLR` environment variable to `1` before process starts, I was imprecise - we need to set it before the process entrypoint starts executing. Windows offers one rarely used way to execute code before entrypoint executes: TLS callbacks. Can we use them to set the environment variable before MSCOREE.DLL starts selecting the CLR runtime to activate? You betcha.

Open a Visual Studio developer command prompt and compile a C# hello world against .NET 3.5:

```csharp
using System;

class Program
{
    static void Main() { Console.WriteLine("Hello world"); }
}
```

```sh
$ csc /noconfig /nostdlib /target:module hello35.cs /r:"C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\v3.5\Profile\Client\mscorlib.dll"
```

The important bit in the above is that we set `target` to `module`. This will produce a `.netmodule` file which is as close as one gets to object files in IL.

Next lets write some C for the TLS callback:

```c
#include <windows.h>

VOID WINAPI tls_callback(
    PVOID DllHandle,
    DWORD Reason,
    PVOID Reserved)
{
    if (Reason == DLL_PROCESS_ATTACH)
        SetEnvironmentVariableW(L"COMPLUS_OnlyUseLatestCLR", L"1");
}

#ifdef _M_AMD64
    #pragma comment (linker, "/INCLUDE:_tls_used")
    #pragma comment (linker, "/INCLUDE:p_tls_callback")
    #pragma const_seg(push)
    #pragma const_seg(".CRT$XLAAA")
        EXTERN_C const PIMAGE_TLS_CALLBACK p_tls_callback = tls_callback;
    #pragma const_seg(pop)
#endif
#ifdef _M_IX86
    #pragma comment (linker, "/INCLUDE:__tls_used")
    #pragma comment (linker, "/INCLUDE:_p_tls_callback")
    #pragma data_seg(push)
    #pragma data_seg(".CRT$XLAAA")
        EXTERN_C PIMAGE_TLS_CALLBACK p_tls_callback = tls_callback;
    #pragma data_seg(pop)
#endif
```

Compile with:

```sh
cl /c hellotls.c
```

Now we just need to merge these together. Mixing C with C# hasn't been a problem since .NET 1 and native tools know how to do that:

```sh
link hello35.netmodule hellotls.obj kernel32.lib /entry:Program.Main /subsystem:console /ltcg
```

I haven't found a way to specify the .NET runtime version of the EXE that link.exe produces, so one last step is to open the produced hello35.exe in a hex editor and search and replace `v4.0.30319` with `v2.0.50727`.

We now have a .NET 3.5 executable that will run on 4.X without additional configuration.
