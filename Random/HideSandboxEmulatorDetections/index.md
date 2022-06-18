---
layout: post
title: "Hide From Sandboxes And Emulators"
---

Most of the #EPP (Endpoint Protection Platforms) products provide some kind of dynamic monitoring capabilities: Sandboxing, Emulation, Hooking, etc.
They monitor when an application calls a library functions ([`CreateFile`](https://docs.microsoft.com/en-us/windows/desktop/api/fileapi/nf-fileapi-createfilew) from `Kernel32`) or use [syscalls](https://en.wikipedia.org/wiki/System_call) (`mov rax, xxx; syscall`) and based on detection logic used by a product the sample is detected or allowed to continue execution.

If your sample uses [`RegSetValue/RegSetValueEx`](https://docs.microsoft.com/en-us/windows/desktop/api/winreg/nf-winreg-regsetvaluea) or lower level [`NtSetValueKey`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nf-wdm-zwsetvaluekey) functions, it's highly likely that a #EPP product you are targeting monitors those calls, because typically they are used to achieve persistence via [Registry](https://en.wikipedia.org/wiki/Windows_Registry).

There is a way to achieve the same goal without using `NtSetValueKey` at all.
`Windows` provides [`Offline Registry Library`](https://docs.microsoft.com/en-us/windows/desktop/DevNotes/offline-registry-library-portal) which can be used to modify a registry hive outside of the active system registry.

We can use `RegSaveKey/RegSaveKeyEx` or `NtSaveKey/NtSaveKeyEx` to save the specified key to a registry file and use [`ORSetValue`](https://docs.microsoft.com/en-us/windows/desktop/DevNotes/orsetvalue) to set a desired value in the offline registry key:

<script src="https://gist.github.com/secrary/c30ec2651e83acff789fe0750f4e3d89.js"></script>

After modifying the offline registry file, calling `RegRestoreKey` function replaces a target key with the modified one from the file:

<script src="https://gist.github.com/secrary/5ed135214b427e3f577af78cd590370e.js"></script>

In the end, the result is the same and the desired value is set without using `NtSetValueKey` function.
It's also less likely that #EPP products monitor `Offline Registry Library` functions.

<img data-src="https://user-images.githubusercontent.com/16405698/58984532-f8055080-87e1-11e9-87b0-3161e8235657.gif" class="lazyload" />

whoami: [@_qaz_qaz](https://twitter.com/_qaz_qaz)