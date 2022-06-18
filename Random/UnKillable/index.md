---
layout: post
title: "make a process unkillable?!"
---


Just start playing with Windows kernel, maybe what I'm writing is foolish, IDK, but found kernel side very interesting.

It's not tutorial, neither trustworthy post, just note for me, maybe full of mistakes, ...that's the only way to improve.

`NOTE: I'm using Windows 10 x64 Version 1709 Build 16299.125`

Let's start from `PsTerminateProcess` function:

![1](https://user-images.githubusercontent.com/16405698/35915989-ca617ae0-0c00-11e8-9010-071f42927bd1.PNG)

It calls `PspTerminateProcess`.

![2](https://user-images.githubusercontent.com/16405698/35915990-ca85bb9e-0c00-11e8-8052-3de20bb4f05a.PNG)

`PspTerminateProcess` calls `PspTerminateThreads`, which traverses all threads and calls `PspTerminateThreadByPointer` for each thread:

![3](https://user-images.githubusercontent.com/16405698/35915991-cad10810-0c00-11e8-9944-ffa2dc78a105.PNG)

`PspTerminateThreadByPointer` calls `KeRequestTerminationThread`:

![4](https://user-images.githubusercontent.com/16405698/35915992-cb027b3e-0c00-11e8-8fc0-6d269c67cdbc.PNG)

`KeRequestTerminationThread` checks 15th bit of 0x74th (`*(v2+116) & 0x4000`) field of `_KTHREAD` and if it set it inserts a kernel mode APC into the APC queue of a thread to kill the thread:

![5](https://user-images.githubusercontent.com/16405698/35915993-cb372258-0c00-11e8-9d2f-fef280767086.PNG)

Seems like if thread is not APC queueable (`15th bit of 0x74 field is not set`) it's impossible to kill a thread (at least this way).

`_KTHREAD` structure's 0x74th field is union, the 15th bit is for `ApcQueueable` flag ([Terminus Project - _KTHREAD](http://terminus.rewolf.pl/terminus/structures/ntdll/_KTHREAD_x64.html)), what if we set this bit to 0?

![6](https://user-images.githubusercontent.com/16405698/35915994-cb7c5fda-0c00-11e8-92b0-bf382344ea1b.PNG)

We can use `WinDbg` or write our driver, driver code is very simple, it receives thread IDs from userland and disables `APCQueueable` flag:

![7](https://user-images.githubusercontent.com/16405698/35915995-cba8023e-0c00-11e8-8e01-5fd2db877cbe.PNG)

DEMO:

<iframe width="560" height="315" src="https://www.youtube.com/embed/EkcN-cT2DBQ?rel=0&amp;controls=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

[YT link](https://www.youtube.com/watch?v=EkcN-cT2DBQ)