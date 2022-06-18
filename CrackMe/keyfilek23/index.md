---
layout: Crackpost
title: "KeyMe by BadSector/k23"
---

The post is about creating `keyfile generator` for `KeyMe by BadSector/k23`

![info](https://user-images.githubusercontent.com/16405698/29776232-0f590bda-8c19-11e7-831a-df50d50bf9b5.png)

Download: [hybrid-analysis](https://www.hybrid-analysis.com/sample/1269373388cb04c5dac76e7cb078d1a9b2ee4be402645404c5fc42e5d409f438?environmentId=100), [VirusTotal](https://virustotal.com/#/file/1269373388cb04c5dac76e7cb078d1a9b2ee4be402645404c5fc42e5d409f438/detection).

**I encourage you to do it yourself before reading the solution.**

It opens `reginf.k23` file and reads `0x24` bytes from it:

![reginf](https://user-images.githubusercontent.com/16405698/29776230-0f565a70-8c19-11e7-9600-bdc9970d1923.png)

In `Check 1` it checks if the first byte contains two same nibbles, for example, `'w'` is same as `0x77` in hex, if so it goes to invalid keyfile message.

In `Check 2` it checks if the second byte is the reverse of first one, for example, if the first byte is `0x64`, second must be `0x46`.

In `Check 3`, the third byte must be sum of first two ones.

In `Check 4`, the fourth byte must be `0`:

![checkz](https://user-images.githubusercontent.com/16405698/29776231-0f57db02-8c19-11e7-8458-53db16201647.png)

We can implement this part of `keyfile generator` in C++:

<script src="https://gist.github.com/secrary/5e5087ae36192a2415311991c5fec094.js"></script>

After that, it modifies middle part of the key (from 5 to 20), in modification it uses the third byte of the key:

![eax](https://user-images.githubusercontent.com/16405698/29777004-925c422a-8c1b-11e7-8d82-b2f0228dab5c.PNG)

We can randomly generate this part:

<script src="https://gist.github.com/secrary/c1983ef930d71d894025acd35c911b2d.js"></script>

It modifies the last part of the key (from 21 to 36), in modification it uses a table of bytes (this table as an array is in `keyfile generator` code), it uses `xlatb` instruction to get a byte from the table:

![table](https://user-images.githubusercontent.com/16405698/29777296-9a88e042-8c1c-11e7-8f60-14a19f31d605.PNG)


After that, it compares results of the last two modifications:

![last](https://user-images.githubusercontent.com/16405698/29777358-d207ce0c-8c1c-11e7-9e4d-784d249ee8c3.PNG)

**What we know:**
- Nibbles in the first byte should not be same.
- The second byte should be reverse of the first one.
- The third is a sum of first two ones.
- The fourth is `0`.
- This is the first part of a key and we can generate this one.
- We can also generate middle part of a key which uses the third byte the first part.
- After modification of the last part of the key, it should be same as middle part after modification, we can brute-force this part and that's exactly what I'm doing in my `keyfile generator`.

Source of the `keyfile generator`:

<script src="https://gist.github.com/secrary/d15e06ea3260ba0fcd971503b89851eb.js"></script>

![keyfile](https://user-images.githubusercontent.com/16405698/29778423-60ae4386-8c20-11e7-942b-d583eb599922.PNG)


Any feedback appreciated.

Twitter: [@_qaz_qaz](https://twitter.com/_qaz_qaz)

