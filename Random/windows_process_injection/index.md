---
layout: rand_post
title: "Windows Process Injection: Poisoned Explorer"
---

***IDEA***

`Explorer` process uses [`LoadLibraryW`](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibraryw) function to load additional libraries at runtime, what if we overwrite the code of `LoadLibraryW` and redirect it to our shellcode?! 
Every time the `Explorer` process calls `LoadLibraryW`, our code will be called without any trigger from the attacker's side.
(We can use any other function, some even more frequently used by the `Explorer` process, also call overwritten function to avoid crashes)

![API_monitor](https://user-images.githubusercontent.com/16405698/69372871-3ec10700-0cab-11ea-9c35-0e4f065add46.PNG)

If our shellcode's size is less than the original code's size, only two process interaction related calls are enough: [`OpenProcess`](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess) and [`WriteProcessMemory`](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory)

***Proof of Concept***

<script src="https://gist.github.com/secrary/c9556033d2487718354d70a0332b7fc8.js"></script>

***DEMO***

<iframe width="640" height="360" src="https://www.youtube-nocookie.com/embed/WuR228Ho3jM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

*whoami*: [@_qaz_qaz](https://twitter.com/_qaz_qaz)



