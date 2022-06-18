---
layout: rand_post
title: "'Injection' Without Injection"
---

[`.shared` sections](https://blogs.msdn.microsoft.com/oldnewthing/20040804-00/?p=38253) are used as a way to share data between multiple instances(e.g. `A'` and `A''`) of an application.
If you modify the shared data in `A'` process, you will get the modified data in `A''` process, even if `A''` is created after the data modification.

`How can we use this to `'inject'` a code?`

If we have an application `'X'`, which contains encrypted/packed code and we want to decrypt the code and execute it from a remote process, after decrypting/unpacking the code, we need to write the code into a remote process (e.g. via `WriteProcessMemory`, etc) and execute it (e.g. via `CreateRemoteThread`, `QueueUserAPC`, etc).

Execution of child process or chain of child processes is a good way to bypass/hinder many security products, but `WriteProcessMemory`, `CreateRemoteThread`, etc. are famous functions used for process injections, so they are suspicious.

`'inject' code using a .shared section`

We can use `.shared` section to modify data in a remote process by just modifying the data inside the current process.

Create a `.shared` section

<script src="https://gist.github.com/secrary/68ab99854b0d65824c8dded7a55d7bb7.js"></script>

* Unpack/decrypt code
* Copy to the shared section
* Execute the current application (the second instance)

<script src="https://gist.github.com/secrary/07fa39822bf7912c8856665ed1ae1672.js"></script>

It's the second instance of the sample application but with modified `.shared` section (there is decrypted/unpacked code)

<script src="https://gist.github.com/secrary/f7eeb8d660cea891ce39bde507ed1bf1.js"></script>

`Conclusion`

The first instance of the application does nothing but decrypting/unpacking a code.
The second instance of the application does nothing but executing the code decrypted/unpacked by the first instance.
This way of separating unpacking and unpacked code execution into two processes without writing the unpacked code to the second instance (e.g. via `WriteProcessMemory`, etc.) maybe can be used to bypass generic unpackers and/or security products.

`PoC`:

<script src="https://gist.github.com/secrary/260c1bf97e1f331d2029e9ea1b612ddf.js"></script>


Any feedback would be greatly appreciated. [@_qaz_qaz](https://twitter.com/_qaz_qaz)