---
layout: rand_post
title: "Abusing WSL for Evasion"
---

[WSL](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux) enables native Linux `ELF64` binaries to run on Windows via the `Windows Subsystem for Linux` (WSL).

![store](https://user-images.githubusercontent.com/16405698/48232534-dce87480-e3a9-11e8-8112-ecbca0380318.png)

From attackers point of view, it's promising, since [`1809`](https://blogs.msdn.microsoft.com/commandline/2018/11/05/whats-new-for-wsl-in-the-windows-10-october-2018-update/) (Windows 10 October 2018 Update) it's possible to install `WSL` distros from the Command Line.

It means that an attacker can enable `WSL` and install a `Linux` distro and execute malicious [`ELF`](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) files in background.

P.S. `C:` drive is mounted on `/mnt/c`

Self-documented `POC`:

The first `Powershell` script (`start.ps1`) enables `WSL` and downloads `Ubuntu1804` package, also registers the task to execute the second script (`resume.ps1`), which installs `Ubuntu` distro and executes the `ELF` file (`encryptDOCX`), all of this happens without any interaction from a user, in background.

<script src="https://gist.github.com/secrary/5ad628cdd1ff93eceb217e58337e9f39.js"></script>

<script src="https://gist.github.com/secrary/d59af177168a15721dbaa7f70b343c3b.js"></script>

I think it's much harder to detect/analyze malicous `ELF` executable on `Windows`.

![procmon](https://user-images.githubusercontent.com/16405698/48232537-dce87480-e3a9-11e8-8158-db85aaa7f474.PNG)


![process_hacker](https://user-images.githubusercontent.com/16405698/48232535-dce87480-e3a9-11e8-8dc6-2ed617829e56.PNG)

Contact: [@_qaz_qaz](https://twitter.com/_qaz_qaz)
