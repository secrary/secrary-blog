---
layout: post
title: "Anti-WinDbg Trick [Quick Post]"
---

If we debug an executable under [WinDbg](http://www.windbg.org/) (Classic and Preview), the debugger will create several distinguishable environment variables for the recently created process.

A sample process without a debugger attached:

![1](https://user-images.githubusercontent.com/16405698/55292768-39dada80-53de-11e9-8f4e-9155e96ac9f7.PNG)

The same sample debugged by [WinDbg Classic](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/):

![2](https://user-images.githubusercontent.com/16405698/55292769-39dada80-53de-11e9-876c-e5110be816bf.PNG)


The same sample debugged by [WinDbg Preview](https://www.microsoft.com/en-us/p/windbg-preview/9pgjgd53tn86):

![3](https://user-images.githubusercontent.com/16405698/55292770-3a737100-53de-11e9-8c94-3cb70ae162dc.PNG)

We can use the change to detect if a sample is under [WinDbg](http://www.windbg.org/) debugger:

<script src="https://gist.github.com/secrary/67a31f75cc9f47f95866da8549706749.js"></script>

![x](https://user-images.githubusercontent.com/16405698/55292882-7d821400-53df-11e9-9df8-1dd94961d7e8.gif)

whoami: [@_qaz_qaz](https://twitter.com/_qaz_qaz)