---
title: Injecting code into executables with C
date: 2007-07-30T11:33:31+00:00
tags:
  - programming
---
In this article, I would like to answer a commonly asked question: is it possible to use my project &#8211; [PE-inject][1] with C? The short answer: yes.

The problem with PE-inject is that it is written in Delphi. Even though a DLL version of PE-inject is available, all the samples are written in Delphi too and for a C programmer, this can be pretty confusing. So, this article will show you how easy it is to use PE-inject with C.

We will create a program which will modify an EXE file in such a way, that the user will be first prompted with a question asking her if she is really sure about running the program. If her answer is Yes, the program will run. Otherwise, it will terminate.

# Creating the injection DLL

First, we need to create a DLL file containing the code to be placed in the EXE files. The [PE-inject documentation][2] says that the library must contain a function called BeforeHandlers or AfterHandlers. We will not talk about the difference between them. For us, the only important thing to know is that when the DLL is injected using PE-inject into an executable, both functions (if present) will be run before the original executable code runs.

```c
#define WIN32_LEAN_AND_MEAN 
#include <windows.h>
#include <winuser.h>

typedef struct _STUB_CONFIGURATION { 
    DWORD NewEntryRVA; 
    DWORD OrgEntryRVA; 
    DWORD ImageBase; 
    DWORD PrefImageBase; 
    DWORD OrgITRVA; 
    DWORD RelocRVA; 
    DWORD MSLRVA; 
    DWORD ExtraDataRVA; 
    DWORD RedirTableRVA; 
    DWORD Flags; 
} STUB_CONFIGURATION, *PSTUB_CONFIGURATION; 

extern "C" __declspec(dllexport) void __stdcall 
BeforeHandlers(PSTUB_CONFIGURATION config) 
{ 
    int action = MessageBox(0, 
        TEXT("Do you really want to execute this program?"), 
        TEXT("Confirm program execution"), 
        MB_YESNO | MB_ICONQUESTION); 
    if (action == IDNO) ExitProcess(0); 
}
```

Save the above source code as injectiondll.cpp. To compile this file, you will also need to create a module definition file containing the list of functions you want to export:

```
LIBRARY	"injectiondll"
EXPORTS
    BeforeHandlers   @1
```

Save it as injectiondll.def to the same directory as injectiondll.cpp. Use the following command to compile the whole code under Visual C++:

```sh
$ cl /MT /LD injectiondll.cpp /link /def:injectiondll.def user32.lib
```

Now you can use the PE-inject Frontend tool (located in the Tools directory of PE-inject) to inject this DLL into any executable file and see the result of your work.

# Using PE-inject programmaticaly

The PE-inject Frontend tool is a nice thing for testing. In the real world however, it would be better to bypass PE-inject Frontend and to create something that would do it&#8217;s job in a more user-friendly way. The InjectFile function located in peinject.dll (part of PE-inject distribution) is exactly what we are looking for.

```c
#include <stdio.h>
#include <windows.h>

#define INJECT_ERR_NOERROR      0
#define INJECT_ERR_FILENOTFOUND 1
#define INJECT_ERR_NOTAPEFILE   2
#define INJECT_ERR_PEHEADERFULL 3

#define INJECT_FLAG_DOIMPORTS   1
#define INJECT_FLAG_JUMPTOOEP   2
#define INJECT_FLAG_HANDLERELOC 4
#define INJECT_FLAG_STRIPRELOCS 8
#define INJECT_FLAG_COMPRESSDLL 16
#define INJECT_FLAG_BACKUPTLS   32

typedef DWORD (__stdcall *INJECTFILEPROC)
    (LPCSTR lpInputFile, LPCSTR lpOutputFile,
    LPCSTR lpDllFile, LPVOID lpExtraData,
    DWORD dwExtraDataSize, DWORD dwFlags);

int main(int argc, char* argv[])
{
    HMODULE hPeinjectDll;
    INJECTFILEPROC InjectFile;
    DWORD result;

    if (argc != 2) {
        printf("Invalid arguments!\n");
        return 1;
    }

    hPeinjectDll = LoadLibrary(TEXT("PEinject.dll"));
    
    if (!hPeinjectDll) {
        printf("Failed to load PEinject.dll!");
        return 2;
    }

    InjectFile =
        (INJECTFILEPROC)GetProcAddress(hPeinjectDll,
        "InjectFile");

    if (!InjectFile) {
        printf("InjectFile() not found!");
        FreeLibrary(hPeinjectDll);
        return 3;
    }

    result = InjectFile(argv[1], argv[1],
        "injectiondll.dll", NULL, 0,
        INJECT_FLAG_DOIMPORTS
        | INJECT_FLAG_JUMPTOOEP
        | INJECT_FLAG_HANDLERELOC
        | INJECT_FLAG_COMPRESSDLL
        | INJECT_FLAG_BACKUPTLS);

    if (!result) {
        printf("File successfuly injected!");
    }
    else
    {
        // one of the INJECT_ERR_ constants
        printf("An error occured!");
    }

    FreeLibrary(hPeinjectDll);

    return 0;
}
```

This code will load the PEinject.dll library (must be in the same directory, or in PATH), locate the InjectFile function and use it to inject the code from injectiondll.dll into executable specified as a command line parameter.

To compile the above code with Visual C++ use this command:

```sh
$ cl injecttest.cpp
```

 [1]: http://migeel.sk/programming/pe-inject
 [2]: http://docs.migeel.sk/PE-inject