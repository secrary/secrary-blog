---
layout: rand_post
title: "Make Your Dynamic Module Unfreeable (Anti-FreeLibrary)"
---

Let's say your product injects a module into a target process, if the target process knows the existence of your module it can call [`FreeLibrary`](https://docs.microsoft.com/en-us/windows/desktop/api/libloaderapi/nf-libloaderapi-freelibrary) function to unload your module (assume that the reference count is one).
One way to stay injected is to hook `FreeLibrary` function and check passed arguments every time the target process calls `FreeLibrary`.

There is a way to get the same result without [hooking](https://en.wikipedia.org/wiki/Hooking).

When a process uses [`FreeLibrary`](https://docs.microsoft.com/en-us/windows/desktop/api/libloaderapi/nf-libloaderapi-freelibrary) to free a loaded module, `FreeLibrary` calls `LdrUnloadDll` which is exported by [`ntdll`](https://www.geoffchappell.com/studies/windows/win32/ntdll/):

![kernel_ntdll](https://user-images.githubusercontent.com/16405698/55341154-a95fd100-5495-11e9-90d9-20b52155774a.PNG)

Inside `LdrUnloadDll` function, it checks the `ProcessStaticImport` field of [`LDR_DATA_TABLE_ENTRY`](https://web.archive.org/web/20190401160724/https://github.com/processhacker/phnt/blob/23b8a7fd449f6e99a7f8b2281d358326b9008ba7/ntldr.h) structure to check if the module is dynamically loaded or not.
The check happens inside `LdrpDecrementNodeLoadCountLockHeld` function:

![call_stack](https://user-images.githubusercontent.com/16405698/55341155-a95fd100-5495-11e9-9c8f-5c105c28bd5b.png)

![ProcessStaticImport_Ghidra](https://user-images.githubusercontent.com/16405698/55341159-a9f86780-5495-11e9-8c99-8eb77c0ae06c.PNG)

If `ProcessStaticImport` field is set, `LdrpDecrementNodeLoadCountLockHeld` returns without freeing the loaded module


![struct](https://user-images.githubusercontent.com/16405698/55341153-a8c73a80-5495-11e9-8c57-9d3c313f88aa.PNG)


So, if we set the `ProcessStaticImport` field, `FreeLibrary` will not be able to unload our module:

![poc](https://user-images.githubusercontent.com/16405698/55341156-a95fd100-5495-11e9-8df4-b8f73cb93223.PNG)

In this case, the module prints `"Hello"` every time it attaches to a process, and `"Bye!"` when it detaches.

**Note:**

There is an officially supported way of doing the same thing:
Calling [`GetModuleHandleExA`](https://docs.microsoft.com/en-us/windows/desktop/api/libloaderapi/nf-libloaderapi-getmodulehandleexa) with `GET_MODULE_HANDLE_EX_FLAG_PIN` flag.
`"The module stays loaded until the process is terminated, no matter how many times FreeLibrary is called."`
Thanks to [`James Forshaw`](https://twitter.com/tiraniddo/status/1112772814803795968)



whoami: [@_qaz_qaz](https://twitter.com/_qaz_qaz)