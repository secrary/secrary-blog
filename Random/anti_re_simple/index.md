---
layout: rand_post
title: "Simple Anti-RE Trick"
---

SHA-256: [7AA84B4CE4FBF937632D3008981C3EF8FF63E1FF846FDBB55060F3973D2507A9](https://www.malware-traffic-analysis.net/2019/07/22/index.html)


Last night I was testing a tracing tool and noticed that the tool crashes when monitoring abovementioned sample, an exception code was `EXCEPTION_ACCESS_VIOLATION` `(0xC0000005)`.
After a bit reversing found that the sample uses a simple anti-RE trick to make an analysis (monitoring, dumping, debugging, etc) bit slower and harder.

***#ifndef trick***


The sample registers `sub_401773` as a [vectored exception handler](https://docs.microsoft.com/en-us/windows/win32/debug/using-a-vectored-exception-handler):

<img data-src="https://user-images.githubusercontent.com/16405698/62903075-14c58380-bd51-11e9-8c30-66a95c7a7514.png" class="lazyload" />

After that, changes the protection value of a recently mapped code (one page) to `PAGE_NOACCESS`:

<img data-src="https://user-images.githubusercontent.com/16405698/62903074-142ced00-bd51-11e9-8f20-a6853026edfb.PNG" class="lazyload" />

Execution continues from the code, accessing to an instruction inside the page triggers `EXCEPTION_ACCESS_VIOLATION` exception and `sub_401773` handler function is called.
`sub_401773` checks type of the exception:

* If the exception is `EXCEPTION_ACCESS_VIOLATION` it changes the protection value of the page to `PAGE_EXECUTE_READWRITE` and executes the instruction again, also sets `TF bit` (single-stepping bit) in [`EFLAGS`](http://www.c-jump.com/CIS77/ASM/Instructions/I77_0070_eflags_bits.htm).
* If the exception is `EXCEPTION_SINGLE_STEP`, it changes the protection value of the page to `PAGE_NOACCESS`:

<img data-src="https://user-images.githubusercontent.com/16405698/62903072-142ced00-bd51-11e9-9ea5-f16479e38c2f.png" class="lazyload" />

So it creates a circle:
* `PAGE_NOACCESS` => `EXCEPTION_ACCESS_VIOLATION` => change to `RWX` and set `TF` => execute an instruction => `EXCEPTION_SINGLE_STEP` => change to `PAGE_NOACCESS` => ...

The code is executed as excepted but the circle makes analysis/monitoring/tracing a bit more challenging.

***#endif***


whoami: [@_qaz_qaz](https://twitter.com/_qaz_qaz)