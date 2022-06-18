---
layout: Crackpost
title: "[A]dvanced Keygenme by  sd333221"
---

CrackMes are a great way to improve reverse engineering skills.
In these series, I'll try to analyze and solve some medium level CrackMes.
Solving similar problems are very helpful for malware analysts.

Today's CrackMe is `[A]dvanced Keygenme by  sd333221`

Date of Release: `2-24-2007`


Download: [hybrid-analysis](https://www.hybrid-analysis.com/sample/3ec77d61d5c72d9065a5b80639f049a4d45a8e630ac333775a83f840afcf67b4?environmentId=100), [VirusTotal](https://virustotal.com/#/file/3ec77d61d5c72d9065a5b80639f049a4d45a8e630ac333775a83f840afcf67b4/detection)

I encourage you to do it yourself before reading the solution.

**Note:** Variables are named by me after analysis.

![crackme](https://user-images.githubusercontent.com/16405698/29512959-e0f0e6d6-8674-11e7-97b7-ef2302ee75b2.PNG)

Thanks to `PeStudio`, now we know the author (`haxx0r`) and fact that the executable uses `Thread Local Storage`.

![pe_studio](https://user-images.githubusercontent.com/16405698/29514086-50855ec0-8678-11e7-9ed0-bbd089adddae.png)

Let's open in `IDA Pro`:

In `tls-callback` it calls `GetTickCount` and computes value of `HIGH_pp` from the result:

![tls-callback](https://user-images.githubusercontent.com/16405698/29514323-2b5c37da-8679-11e7-92b4-11c770f008ca.png)

In `GetRand_CPUID_getTick` it gets CPU vendor ID using `cpuid` (EAX=0) , and uses `HIGH_pp` to compute new value:

![GetRand_CPUID_getTick](https://user-images.githubusercontent.com/16405698/29514447-a2c0e4ba-8679-11e7-96b1-7fc14fe043aa.png)

After that, it converts the result to `string` using `_itoa`.

`modify_str` contains two parts `mix_two_list` and `modify_cpuid_time`:

![image](https://user-images.githubusercontent.com/16405698/29517883-2d8f5ece-8688-11e7-94bc-f6a6c66e2865.png)

In `mix_two_list` it builds a list of bytes(I renamed variable as `alpha`) from `0x0` to `0xFF` and modifies it using another list of `13337....133371`:

![image](https://user-images.githubusercontent.com/16405698/29517876-20411014-8688-11e7-9e81-5ec90e97fb8e.png)

Python implementation of `mix_two_list` function:

<script src="https://gist.github.com/secrary/11f2ee3133f8bb560c54b858dc93bde3.js"></script>

In `modify_cpuid_time` it uses the value converted from integer to string and modifies this value using `alpha` list:

![image](https://user-images.githubusercontent.com/16405698/29518058-ff40cea8-8688-11e7-963a-7c4d5c919b8d.png)

Python implementation of `modify_cpuid_time`:

<script src="https://gist.github.com/secrary/243aa7272c3f42a600fe1ebbeb18b6dd.js"></script>

`modify_str` is used to encrypt/decrypt computed value before using it.

At the end of `tls-callback`, it uses two ways to detect debuggers, named as `chckDbgr` and `anotherCheck`.
`chckDbgr` uses  "guard" pages to detect debuggers and `anotherCheck` uses `FindWindow()` and `CreateFile()` functions, both tricks are explained in [`The Ultimate Anti-Reversing Reference by Peter Ferrie`](https://encrypted.google.com/search?safe=active&hl=en&q=The+Ultimate+Anti-Reversing+Reference+by+Peter+Ferrie) (must read!)

![image](https://user-images.githubusercontent.com/16405698/29518720-86b67e1c-868b-11e7-8b99-419c0246beb6.png)

The CrackMe uses custom base64 alphabet: `/+9876543210zyxwvutsrqponmlkjihgfedcbaZYXWVUTSRQPONMLKJIHGFEDCBA` but if it detects a debugger it calls `ifDebugg__` function, which reverses the alphabet:

![image](https://user-images.githubusercontent.com/16405698/29518833-08cd2400-868c-11e7-8741-5145fea39f88.png)

That's `tls-callback`, in `WinMain` it sets the result of our old friend `GetRand_CPUID_getTick` as username, that's exactly same value what we have in encrypted form.

![image](https://user-images.githubusercontent.com/16405698/29518933-7652d8ee-868c-11e7-84eb-301819e1843b.png)


Let's analyze `CheckSerial` function (we are trying to write keygen, don't patch it)
![image](https://user-images.githubusercontent.com/16405698/29519055-f2740ee8-868c-11e7-8e18-50637da30570.png)

In `CheckSerial` function it decrypts the value (Yeah, this is from `tls-callback`), decrypted version of text is same as `username`, both are from `GetRand_CPUID_getTick`, after decryption it encodes result with `base64` but uses custom alphabet, encrypts result once again and encodes result as well, final result is saved in `esi` register

![image](https://user-images.githubusercontent.com/16405698/29519346-252470c0-868e-11e7-9c65-1023caf3499c.png)

`SerialComp` gets our input as a parameter and returns value which should be same as in `esi`.

`SerialComp` is some kind of decoding function:

![image](https://user-images.githubusercontent.com/16405698/29519634-2e7b6484-868f-11e7-8381-cc45725d081e.png)

Python REVERSE implementation of `SerialComp`:

<script src="https://gist.github.com/secrary/2a3dfac9fcfb4ae45985f818734b4958.js"></script>

**What we know:**
- It gets value from `cpuid` and `GetTickCount` - same as username
- Encodes the value with `base64` using custom alphabet
- Encrypts decoded value
- Encodes encrypted value

Main parts of writing keygen are implementing reverse of `SerialComp` and encryption function (I'm using the built-in support of `base64`).

We have everything we need to write keygen:

<script src="https://gist.github.com/secrary/0e747196018eecb2f10ee71ce42ecad3.js"></script>

Any feedback appreciated.

Twitter: [@_qaz_qaz](https://twitter.com/_qaz_qaz)