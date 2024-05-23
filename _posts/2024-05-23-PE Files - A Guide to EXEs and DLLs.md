---
title: PE Files - A Guide to EXEs and DLLs
tags: 
categories: MalDev
---

Welcome to my Malware development series on my blog. I'm just starting to learn so I'm going to share what I learn here. I would like to emphasize that I am providing this information in an educational way and always supporting the ethical way.
From this post I will create more content being this the first one :) nothing more to add, so here we go!

# Portable Executable

## What is PE?

The Portable Executable (PE) format is the standard file format for EXE files and Dynamic Link Libraries (DLL) used in Windows OS

### EXE vs DLL

EXEs are sepate programs that can be <ins>loaded into memory as independent process</ins>.

DLLs are modules that are <ins>loaded into a process</ins> and cannot be ran independently in memory.

### PE structure

It is mainly divided into 2 parts: Headers and sections:

![1780b0d9400766bd0bb31a2247fae0ee.png](/assets/img/screenshots/pe1/8dbe616191e70f1edb3f419c2e29fe0a.png)

So the Windows loader loads the PE file into memory, checks the headers, checks dependencies, and establishes the execution environment for the PE file.

## Generating and analysing  a PE file

To analyse executable, tools like [PE-Bear](https://github.com/hasherezade/pe-bear), [Process Hacker](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://processhacker.sourceforge.io/&ved=2ahUKEwiolJK0rZSGAxUGU6QEHZ4KBEoQFnoECCMQAQ&usg=AOvVaw0gTZxS5yGpmBHzPpQiXeD9), your chosen IDE, along with a compiler such as [GCC](https://gcc.gnu.org/), are required.

### Compilation

To create our first PE, we will need to translate C++ code to an understandable language. In my case i will be using [GCC](https://gcc.gnu.org/).

Essentially, compilation translates human-readable source code into machine-readable object code:

1.  **Preprocessing**: Handles directives like `#include` and macro definitions.
2.  **Compilation**: Translates preprocessed code into assembly language.
3.  **Assembly**: Converts assembly code into machine code (object files).
4.  **Linking**: Combines object files into a single executable (PE file).

![1780b0d9400766bd0bb31a2247fae0ee.png](/assets/img/screenshots/pe1/1780b0d9400766bd0bb31a2247fae0ee.png)

### Hands on! (EXE)

The first thing to create the exe will be opening the IDE (in my case Visual Studio Code) and write a simple hello world in C++. An executable typically has a `main()` function as its entry point, which is called when the program starts:

```
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <iostream>

int main(void){
    printf("Hello World!");
    getchar();
    return 0;
}
```

If you are using VS, build the code to compile it, and an .exe file will be created in the same folder as the .cpp file.

After compiling the code, run the .exe file. It will open the console, print "Hello World!", and wait for a key press before exiting.

While the executable is running (do not close the console), open Process Hacker and look for the file name, in this case, HW.exe. Process manageris a powerful tool that provides advanced features beyond the standard Task Manager.

![63cc54813785ca81e4e12aeef9b4ea37.png](/assets/img/screenshots/pe1/63cc54813785ca81e4e12aeef9b4ea37.png)

In the module stab we can search for al the .dll loaded in memory of the process:

![b69b345d428411770973715c5bacad23.png](/assets/img/screenshots/pe1/b69b345d428411770973715c5bacad23.png)

In the "Memory" tab you can find how memory is utilized by the selected process:

![ea7bf398fefdf47d0332209881f19d70.png](/assets/img/screenshots/pe1/ea7bf398fefdf47d0332209881f19d70.png)

### Hands on! (DLL)

Now wee will create a DLL to see the difference. The next step will be creating a simple Hello World in C++.  In a DLL "HelloWorld" code, you define a function that can be exported from the DLL. This function is typically called by another program (e.g., an EXE) that loads the DLL. The purpose of a DLL "Hello World" code is to provide a reusable piece of functionality that can be shared among multiple applications.

A DLL does not have a predefined entry point like `main()`. Instead, it exports functions that can be called by other programs.

```
// HelloWorldDLL.cpp : Defines the exported functions for the DLL.
#include <windows.h>
#include <pch.h>

// Exported function declaration
extern "C" __declspec(dllexport) void HelloWorld();

// Exported function definition
void HelloWorld() {
    MessageBoxA(0, "Hello, World!", "DLL Message", MB_OK | MB_ICONINFORMATION);
}

// Optional: DllMain entry point (usually not necessary for simple DLLs)
BOOL APIENTRY DllMain(HMODULE hModule, DWORD  ul_reason_for_call, LPVOID lpReserved) {
    switch (ul_reason_for_call) {
    case DLL_PROCESS_ATTACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

After compiling the code in VS, a .dll will be generated. IN this case I will use rundll32 from CMD to run it:

`rundll32 .\HW.dll,HelloWorld`

![084ad9d734a970c55c30dd5148c8fc9a.png](/assets/img/screenshots/pe1/084ad9d734a970c55c30dd5148c8fc9a.png)

After running the program, you'll encounter a "Hello World" message box. Next, open Process Hacker and search for the name you assigned to the DLL file, such as "HW.dll". Upon inspection, you'll notice that Process Hacker detects a process named `rundll32.exe` in execution. This occurrence is expected, as `rundll32.exe` serves as the program responsible for loading and executing the DLL file.

![7f746cefeb9e85b21956c266dc5a4dc6.png](/assets/img/screenshots/pe1/7f746cefeb9e85b21956c266dc5a4dc6.png)

If we open the process and delve deeper into the Modules tab, we'll observe that `HW.dll` is indeed loaded into the process.

![fbd4349400ce47b12920ccf1908a0093.png](/assets/img/screenshots/pe1/fbd4349400ce47b12920ccf1908a0093.png)

And that wraps up today's post. In the next posts, I'll show how to embed payloads in PE files and creating a backdoor in a legitimate application, bypassing Windows Defender, etc.