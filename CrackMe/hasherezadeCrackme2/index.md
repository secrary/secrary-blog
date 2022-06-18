---
layout: post
title: "Malwarebytes CrackMe 2 by hasherazade"
---

Tha crackme is created by [@hasherazade](https://twitter.com/hasherezade) for malware analysts.

You can download it from [here](https://blog.malwarebytes.com/security-world/2018/04/malwarebytes-crackme-2-another-challenge/)

![crackme](https://user-images.githubusercontent.com/16405698/39401250-499ea15c-4b30-11e8-84ba-76cd1304bcd7.PNG)

# Environment and tools used

For the analysis environment, I used [Windows 10 64-bit](https://en.wikipedia.org/wiki/Windows_10) on [VMware Workstation Pro](https://www.vmware.com/products/workstation-pro.html), with an Internet connection.

During the analysis, I used the following tools:

* [IDA Pro](https://www.hex-rays.com/products/ida/index.shtml)
* [x64dbg/x32dbg](https://x64dbg.com/)
* [flare-floss](https://github.com/fireeye/flare-floss)
* [python-exe-unpacker](https://github.com/countercept/python-exe-unpacker)
* [010 Editor](https://www.sweetscape.com/010editor/)
* [Visual Studio Code](https://code.visualstudio.com/)
* [WinDbg](http://windbg.org/)

# Initial analysis

From static analysis we see that it's a python based program bundled by [`PyInstaller`](http://www.pyinstaller.org/) or [`Py2exe`](http://py2exe.org/) (two of the most popular tools).

![flare_floss](https://user-images.githubusercontent.com/16405698/39463175-f0e16036-4d05-11e8-9251-66c745b1a0ac.PNG)

`NOTE:` I recommend you to use [`flare-floss`](https://github.com/fireeye/flare-floss) instead of a `strings` tool.

We can use [`python-exe-unpacker`](https://github.com/countercept/python-exe-unpacker) tool to unpack the `crackme`. `python-exe-unpacker` is just the wrapper around [`unpy2exe`](https://github.com/matiasb/unpy2exe), [`pyinstxtractor.py`](https://www.aldeid.com/wiki/Pyinstxtractor) and [`uncompyle6`](https://github.com/rocky/python-uncompyle6).

![3](https://user-images.githubusercontent.com/16405698/39401252-49f2e3c0-4b30-11e8-8891-dd5fabe917a3.PNG)

The main file is one with no extension: `another`, but that's not the plain source code, it's the compiled one.

![4](https://user-images.githubusercontent.com/16405698/39401239-47f5cf88-4b30-11e8-9777-2adc50b0e16c.PNG)

We can decompile it with `uncompyle6`. Occasionally, the main python file that is packed with `PyInstaller` does not contain the magic number - `0x03 0xF3 0x0D 0x0A 0x00 0x00 0x00 0x00`, we need to append the magic number (if there is none) and decompile the file using `uncompyle6`:

![5](https://user-images.githubusercontent.com/16405698/39401240-481d9130-4b30-11e8-9359-3af5037e6983.png)

![6](https://user-images.githubusercontent.com/16405698/39401241-4851db70-4b30-11e8-8b76-f3b8eecc40b4.png)

`NOTE: we need to append ".pyc" extension, otherwise "uncompyle6" complains`

Now we have the source code:

![7](https://user-images.githubusercontent.com/16405698/39401242-4876f360-4b30-11e8-942f-4e623dc6a0b0.PNG)

# `Stage 1`

At the first stage, it checks login, password, and PIN.
We clearly see that `login` is `hackerman` and the password's `MD5` hash is `42f749ade7f9e195bf475f37a44cafcb`, we can use an online decryptor tool to get the password:

![8](https://user-images.githubusercontent.com/16405698/39401243-489c24f0-4b30-11e8-9e36-a37cfd8c3035.PNG)

![9](https://user-images.githubusercontent.com/16405698/39401244-48c27d4e-4b30-11e8-94ec-cfbf8f56050e.PNG)

It uses `PIN` to generate `key` value and checks `MD5` hash for the value, at online databases there is no entry for `fb4b322c518e9f6a52af906e32aee955`, so we need to brute-force one (assume that `PIN` is 4 bytes long and only numeric values from 0 to 9):

![10](https://user-images.githubusercontent.com/16405698/39401245-48e8d05c-4b30-11e8-843d-fb2293c522b7.PNG)

My script to brute-force `PIN`:

<script src="https://gist.github.com/secrary/c3279cae598622fc94050f561acf0494.js"></script>

`Stage 1` solved:

`login:` hackerman

`password`: Password123

`PIN`: 9667

![11](https://user-images.githubusercontent.com/16405698/39401246-490ddb2c-4b30-11e8-9504-db2ce5b4d37d.PNG)

# `Stage 2`

At `decode_and_fetch_url` it decrypts the URL and downloads the image:

![12](https://user-images.githubusercontent.com/16405698/39401247-49347516-4b30-11e8-867f-fafd70dae385.PNG)

It uses `get_encoded_data` to extract data from the image, and `is_valid_payl` to check if the data is PE executable:

![13](https://user-images.githubusercontent.com/16405698/39401248-4957960e-4b30-11e8-86e1-009dfd3c2063.PNG)

After that, it allocates memory, copies the data and jumps to `allocated_mem + 2` location via `call` instruction (seems like the downloaded file contains shellcode functionality):

![14](https://user-images.githubusercontent.com/16405698/39401249-497baa9e-4b30-11e8-8f91-614586db2777.PNG)

We need to analyze the decrypted file. Just dump the PE file while debugging the source.

The shellcode starts from the third byte of the file:

`NOTE: base address of the file might be different on different screenshots. The important part is RVAs`.

![1](https://user-images.githubusercontent.com/16405698/39405019-b0623b90-4b8c-11e8-82ed-02560ef14c16.png)

To debug the second stage file, we pause debugging source code at `MR()` call (transfer call), attach a debugger to the `python.exe` process and set `EIP` to allocated memory:

![2](https://user-images.githubusercontent.com/16405698/39405020-b088ccce-4b8c-11e8-8d26-88754cfab847.png)

Instructions at the beginning are used to jump to `base+0x6E0` location:

![3](https://user-images.githubusercontent.com/16405698/39405021-b0b5b3ba-4b8c-11e8-99c0-4fb02884d915.png)

`NOTE: 0x7BA0000 is the base address of the file in the memory`

At `base+0x6E0` it builds [`IAT (import address table )`](https://en.wikipedia.org/wiki/Portable_Executable)

![4](https://user-images.githubusercontent.com/16405698/39405022-b0d90bd0-4b8c-11e8-9d3c-7de02edd7e88.PNG)

After getting all necessary function addresses, it jumps to [`OEP (Original Entry Point)`](https://www.aldeid.com/wiki/OEP-Original-Entry-Point) of the PE file (it's `.dll`) via `call esi` at the near end of `base+0x6E0` function:

![5](https://user-images.githubusercontent.com/16405698/39405023-b1007e86-4b8c-11e8-8428-10ea4d3018e6.PNG)

There is tons of unnecessary code, the most important part for us, starts at `base+0x1170`:

![6](https://user-images.githubusercontent.com/16405698/39405024-b125c7ae-4b8c-11e8-8765-cd07afaa2b17.png)

It registers two [vectored exception handlers](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681411(v=vs.85).aspx), `VEC_Handler_set_enviroment_var` and `handler_VEC_getEnvValue`.

`VEC_Handler_set_enviroment_var` sets the current process ID as `mb_chall` environment variable for the process and returns `0 - EXCEPTION_CONTINUE_SEARCH`

![7](https://user-images.githubusercontent.com/16405698/39405025-b1495fca-4b8c-11e8-872c-73993434127f.PNG)

`handler_VEC_getEnvValue` increases `EIP - instruction pointer` by 6 (a.k.a skip next 6 bytes of instructions) and returns `0xFFFFFFFF - EXCEPTION_CONTINUE_EXECUTION`

![8](https://user-images.githubusercontent.com/16405698/39405026-b16fe9ce-4b8c-11e8-9928-52f872d97bc7.PNG)

There is an anti-debug trick using `INT 3 (0xCC)` instruction.

# INT 3 trick

Let's say we have following code:

![9](https://user-images.githubusercontent.com/16405698/39405027-b195f3c6-4b8c-11e8-99c1-9da3a2bc62a2.PNG)

When the [`INT3 - EXCEPTION_BREAKPOINT`](https://en.wikipedia.org/wiki/INT_%28x86_instruction%29) occurs an attached debugger has the chance to handle the exception, if there is no a debugger attached, registered [`VEH` and `SEH`](https://en.wikipedia.org/wiki/Microsoft-specific_exception_handling_mechanisms) executes, if any debugger implicitly handles the exception, it just skips one instruction `0xCC` and continues execution, we can use this method to change our program behavior under some debuggers.

`WinDbg` implicitly handles `INT 3` exception:

![10](https://user-images.githubusercontent.com/16405698/39405028-b1be8804-4b8c-11e8-91ad-fbc19fbcba73.PNG)

If there is no a debugger attached, which handles the exception, we can use the `VEH` or `SEH` handler to change the code execution path, by modifying `EIP` register (as shown at the code snippet above)

`IDA Pro` gives us the chance to decide which way is preferable to us:

![11](https://user-images.githubusercontent.com/16405698/39405008-aeb0930a-4b8c-11e8-8e34-833a6eee8d36.PNG)

In the `crackme`, `INT 3` is used to execute the `msg_sorry_failed` function if the execution happens under a debugger:

![12](https://user-images.githubusercontent.com/16405698/39405009-aed80bf6-4b8c-11e8-995f-aafd3ccdeb0c.PNG)

![13](https://user-images.githubusercontent.com/16405698/39405010-af016708-4b8c-11e8-8570-f7aa46ca673c.PNG)

If there is no debugger, which handles the exception, the `crackme` skips the next 6 bytes and continues execution from `enum_windows__HERE` function, which creates a new thread and executes `enumWnd_10001110` function:

![14](https://user-images.githubusercontent.com/16405698/39405011-af28c1ea-4b8c-11e8-80fc-af14cf8b2db7.PNG)

`EnumWindows` calls `enumWindProc` for each top-level window and checks if a window name contains `Notepad` and `secret_console` strings:

![1](https://user-images.githubusercontent.com/16405698/39406936-2c7b5128-4bae-11e8-86d6-388752053d0f.PNG)

If so, it changes the window title to `"Secret Console is waiting for the commands..."`

![2](https://user-images.githubusercontent.com/16405698/39406937-2c9e9eee-4bae-11e8-941f-8acda98d6bc1.PNG)

After that, it enumerates child windows and checks if the content contains the `dump_the_key` string.

![3](https://user-images.githubusercontent.com/16405698/39406935-2c48599e-4bae-11e8-916c-8757f621f297.PNG)

If so, it loads `actxprxy.dll` and overwrites the first 0x269 bytes with encoded/encrypted/compressed data:

![19](https://user-images.githubusercontent.com/16405698/39405017-b01696ae-4b8c-11e8-8e19-79b127b56599.PNG)

![overwrite_mz](https://user-images.githubusercontent.com/16405698/39405001-8c429250-4b8c-11e8-8fc7-2c859ce7d3a1.gif)

`NOTE:` click [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ff381405(v=vs.85).aspx) to read more about `Windows messages`.

We did it. `Stage 2` solved:

![20](https://user-images.githubusercontent.com/16405698/39405018-b039432a-4b8c-11e8-9d3a-e1ef1b47a441.PNG)

# `Stage 3`

Gets address of the recently loaded module, calculates and checks the checksum, decompresses the overwritten data and `XOR`s with a user entered key:

![1](https://user-images.githubusercontent.com/16405698/39405278-eeab3398-4b91-11e8-9ddb-ea77b2eecd80.PNG)

`level3_colors` is used to get a key from a user in form of an `R G B` color code:

![2](https://user-images.githubusercontent.com/16405698/39405277-ee852a54-4b91-11e8-8eea-def601da2de6.PNG)

After decoding the data, it tries to execute the data using [`exec`](https://docs.python.org/2.0/ref/exec.html) command, the `exec` method executes the dynamically created program, which is either a string or a code object, simple example:

<script src="https://gist.github.com/secrary/dfdbc15a1afe89458553424bad868757.js"></script>

The challenge is that we should guess correct the `R G B` sequence to decrypt the data and execute it.

My brute-force script is the following:

<script src="https://gist.github.com/secrary/22b0feadc5d587c34141526d75478acf.js"></script>

The `data` variable is part of the encrypted data (there is no need to decrypt whole data to guess the correct sequence), and it the checks if the result is a printable string:

![3](https://user-images.githubusercontent.com/16405698/39405491-6e1a1592-4b95-11e8-9bd2-0fb85bc7cb2b.PNG)

`Stage 3` solved: `R` - 128, `B` - 0, `G` - 128

![all](https://user-images.githubusercontent.com/16405698/39405252-850e00b4-4b91-11e8-90bf-401b0a17ef83.gif)

Thank you [@hasherezade](https://twitter.com/hasherezade) for such an interesting crackme.

Any feedback would be greatly appreciated.

Twitter: [@_qaz_qaz](https://twitter.com/_qaz_qaz)

Discuss on [Reddit](https://www.reddit.com/r/ReverseEngineering/comments/8kjqn8/how_to_solve_the_malwarebytes_crackme_2_a/)