---
layout: post
title: "Unexported Windows kernel functions/structures finding method"
---

Many functions and structures are not exported by `nt`, such as `PsGetNextProcess` function, `KeServiceDescriptorTable` and many others.

How can we get virtual addresses of desired functions and/or structures?

There are methods which use pattern matching to find specific functions and/or structures inside that function, but this way of finding is unreliable (due to changes from `MS` can break our pattern matching algorithm).

What about using [`Debug Help Library`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms679291(v=vs.85).aspx) from Microsoft? We can access the symbolic debugging information of an image, such as `%systemroot%/system32/ntoskrnl.exe`, extract `RVA` for desired function/structure and add to address of `nt`.

I'm using `EnumDeviceDrivers` from `Psapi` to get an address of `ntoskrnl.exe`, and [`SymFromName`](https://www.codeproject.com/articles/132742/writing-windows-debugger-part) to get symbolic information of a function/structure.

I'm assuming that target system does not contain any debugging related executables, such as `symchk.exe`, `SymSrv.dll`, etc.

To get `.pdb` file, which contains debugging inforamtion for `ntoskrnl.exe` we need to download it [manually using `symchk.exe`](https://msdn.microsoft.com/en-us/library/windows/desktop/ee416588(v=vs.85).aspx#getting_symbols_manually):

![1](https://user-images.githubusercontent.com/16405698/36641180-37f6f74e-1a23-11e8-96a5-ef99e400f104.PNG)

![CreateProcess](https://user-images.githubusercontent.com/16405698/36643635-560bd4fc-1a46-11e8-94c5-c3a75ca6b57e.PNG)

We can find `symchk.exe` under `C:\Program Files (x86)\Windows Kits\10\Debuggers\x64` on Windows 10, `symchk.exe` uses `SymbolCheck.dll`, `SymSrv.dll` and `DbgHelp.dll`.

![4](https://user-images.githubusercontent.com/16405698/36641177-3792d412-1a23-11e8-93e1-bef1b983bf19.PNG)

It's a good idea to embed all necessary files into main executable and extract them at run-time, we need following additional executables: `DbgHelp.dll`, `SymbolCheck.dll`, `symchk.exe` and `SymSrv.dll`

![5](https://user-images.githubusercontent.com/16405698/36641594-f54501f0-1a29-11e8-85f1-2329807243bb.PNG)


Example source code:

<script src="https://gist.github.com/anonymous/83638b317ae735b6d9a779f4addad863.js"></script>


![6](https://user-images.githubusercontent.com/16405698/36641179-37d32cc4-1a23-11e8-88c0-9c699507cd1c.PNG)

Advantages of this method:

* Under right circumstances, we get accurate information.
* Cross-platform ?

Disadvantages of this method:

* We need user-mode process
* We need Internet connection
* Size of user-mode application is quite large due to it contains several executables.


Thank you for your time...

Twitter: [@_qaz_qaz](https://twitter.com/_qaz_qaz)