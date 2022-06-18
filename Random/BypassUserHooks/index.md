---
layout: post
title: "Bypasss User-Mode Hooks"
---

User-mode hooks are unreliable and there are tons of ways to bypass them, for example, [`makin`](https://github.com/secrary/makin) loads `ntdll` from the `%temp%` directory and bypasses all hooks from original `ntdll`, but it loads DLL, so it's noisy. What about using `ntdll` level functions? it's better than using `KernelBase` and other higher level DLLs but still easy to hook.

Today I want to talk about another method, which I think is hardest one to hook from user mode - reimplementing `ntdll` functions.

To become more stealthy we need to go deeper, use undocumented functions, which makes our methods Windows version depended.

`NOTE: Windows Version 1709 x64`

We can use IDA or any other disassembler to rewrite functions.

`NtCreateFile`:

![NtCreateFile](https://user-images.githubusercontent.com/16405698/36338625-8565857c-13ac-11e8-8135-3f1ea1046a54.PNG)

`NtClose`:

![NtClose](https://user-images.githubusercontent.com/16405698/36338624-853de38c-13ac-11e8-88bf-1544b0ccd838.PNG)

Main:

<script src="https://gist.github.com/anonymous/65fce3db8af2260edee9a26ccf3157d9.js"></script>

`NOTE:` for more stability, you can extract index number from `ntdll` at runtime:

![ntdll](https://user-images.githubusercontent.com/16405698/36371209-53beda4c-1559-11e8-9bbc-f07bb5f05978.PNG)


Download the source code from [here](https://github.com/secrary/sources_from_secrary_posts/tree/master/BypassUserHooks).

DEMO:

![bypasshookdemo](https://user-images.githubusercontent.com/16405698/36338650-f2c2b5fe-13ac-11e8-9c38-401ff014e087.gif)

(click [here](https://user-images.githubusercontent.com/16405698/36338650-f2c2b5fe-13ac-11e8-9c38-401ff014e087.gif) to view a larger version)


Twitter: [@_qaz_qaz](https://twitter.com/_qaz_qaz)