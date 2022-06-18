---
layout: rand_post
title: "Simple But Effective Anti-Sandbox Trick"
---

![code_example](https://user-images.githubusercontent.com/16405698/63274873-141c7800-c290-11e9-8f57-fc78ba524aff.png)


We can modify VBA Macro notification settings from the Registry by creating `VBAWarnings` DWORD under 
`HKEY_CURRENT_USER\software\policies\microsoft\office\{ms_office_version}\{application}\security`.

Possible values for `VBAWarnings`:
- Value 1: **Enable All Macros**
- Value 2: Disable All macros with notification
- Value 3: Disable all macros except those digitally signed
- Value 4: Disable all without notification

When opening a document with a macro, MS Office application (`winword.exe`, etc)  tries to access `VBAWarnings` value:
<img data-src="https://user-images.githubusercontent.com/16405698/63274217-f39fee00-c28e-11e9-8dc8-abfd46a98b1a.PNG" class="lazyload" />

All online sandbox services I've tested use the feature to enable all macros without any notification (value 1), although normal users usually don't have the feature enabled.

## **Any.Run**
<img data-src="https://user-images.githubusercontent.com/16405698/63274221-f4388480-c28e-11e9-8df3-63ed0615e873.png" class="lazyload" />

## **Hybrid-Analysis**
<img data-src="https://user-images.githubusercontent.com/16405698/63274219-f4388480-c28e-11e9-8d53-2be6bb869035.png" class="lazyload" />



whoami: [@_qaz_qaz](https://twitter.com/_qaz_qaz)