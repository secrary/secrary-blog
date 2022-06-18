---
layout: rand_post
title: "Is there a hidden process?"
---

When Windows creates a process, at kernel side `NtCreateUserProcess` calls `PspAllocateProcess`, which calls `ObCreateObjectEx` with `PsProcessType` as object type parameter:

![2](https://user-images.githubusercontent.com/16405698/37045896-e651cfca-215e-11e8-9bf0-be8ca70161ef.PNG)

![3](https://user-images.githubusercontent.com/16405698/37045897-e67ccd7e-215e-11e8-9926-62d678a82e88.PNG)

`PsProcessType` is the instance of `_OBJECT_TYPE`:

![1](https://user-images.githubusercontent.com/16405698/37045895-e626f156-215e-11e8-9a71-731cd4f59652.PNG)

Seems like `TotalNumberOfObjects` field of `_OBJECT_TYPE` refers to the number of total objects, in our case, it's a number of processes.

We can get a list of processes via parsing `ActiveProcessLinks` and compare it to `TotalNumberOfObjects` field.

![4](https://user-images.githubusercontent.com/16405698/37045899-e6b01972-215e-11e8-930c-3edf4c00ebd0.PNG)

This way we can detect if there is a hidden process, but not which one.

Any feedback appreciated: [@_qaz_qaz](https://twitter.com/_qaz_qaz)