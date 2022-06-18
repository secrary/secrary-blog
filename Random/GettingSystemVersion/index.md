---
layout: post
title: "Getting the System Version [UNSTABLE]"
---


[Windows API Sets](https://msdn.microsoft.com/en-us/library/windows/desktop/hh802935(v=vs.85).aspx) mechanism is introduced by Microsoft from Windows 7. Also, `PEB` structure has new field `ApiSetMap`:

![1](https://user-images.githubusercontent.com/16405698/36369133-0bc5b7d0-1552-11e8-9e59-c04163edee58.PNG)

`ApiSetMap` is a pointer to another structure `_API_SET_NAMESPACE`:

![2](https://user-images.githubusercontent.com/16405698/36369135-0bf5cb78-1552-11e8-94b9-178972910e95.PNG)

According to [lucasg's post](https://lucasg.github.io/2017/10/15/Api-set-resolution/) `Version` value is different on Windows 7, Windows 8.1 and Windows 10, which means we can use this field to get Windows system version:

![3](https://user-images.githubusercontent.com/16405698/36369137-0c21d682-1552-11e8-9394-33147c8b9ced.PNG)


Example:
<script src="https://gist.github.com/anonymous/ece25ec04543d091e7fa86de8886f63d.js"></script>


NOTE: It's not recommended to use this method in a production product, there is a much stable way from [`MSDN`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724429(v=vs.85).aspx) using `Version API Helper functions`;