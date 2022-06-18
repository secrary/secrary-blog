---
layout: Crackpost
title: "Imagination by kratorius"
---

According to [Wikipedia](https://en.wikipedia.org/wiki/Crackme), `a crackme is a small program designed to test a programmer's reverse engineering skills.`

![1](https://user-images.githubusercontent.com/16405698/37252454-e476317e-2518-11e8-8aaf-4d76a978540a.PNG)

Today I want to write about a medium level crackme from `crackmes.de` archive called: `Imagination`

SHA1: `C052CDAD49297F854E832208AFB7CAB8D637C870`

![2](https://user-images.githubusercontent.com/16405698/37252436-e1983664-2518-11e8-966a-2dd60f49e98d.PNG)

Okay, there is no username/password pair, we need to satisfy its requests.

![4](https://user-images.githubusercontent.com/16405698/37252438-e1e455bc-2518-11e8-88eb-54607bb69c7d.PNG)

After clicking the `Unlock Me` button, the function at `0x0401470` executes, it tries to open the `ohmygod.bmp` file from the current directory of the `crackeme`

![5](https://user-images.githubusercontent.com/16405698/37252439-e20e25d6-2518-11e8-92c7-3142a0cf60ae.PNG)

`NOTE`: [Ange Albertini](https://twitter.com/angealbertini)'s poster about `BMP` is very helpful if you don't know anything about `BMP` file structure like me.

![10](https://user-images.githubusercontent.com/16405698/37252444-e30a559a-2518-11e8-8f54-45d2bed4719f.png)

After successfully opening the file it calls the function at `0x0401040` (renamed by me as `parse_header_0x0401040`)

![6](https://user-images.githubusercontent.com/16405698/37252440-e2370384-2518-11e8-8821-4f54b63d3709.PNG)

It reads 14 bytes and 40 bytes from the file via calling `ReadFile` two times, according to [`MSDN`](https://msdn.microsoft.com/en-us/), 14 bytes is the size of a [`BITMAPFILEHEADER`](https://msdn.microsoft.com/en-us/library/windows/desktop/dd183374(v=vs.85).aspx) structure and 40 bytes is the size of a [`BITMAPINFOHEADER`](https://msdn.microsoft.com/en-us/library/windows/desktop/dd183376(v=vs.85).aspx) structure:

![7](https://user-images.githubusercontent.com/16405698/37252441-e25c913a-2518-11e8-885a-81cfcc793b8e.PNG)

After that there are several checks of fields: `btType`, `biBitCount`, `bfSize` and `biCompression`:

![8](https://user-images.githubusercontent.com/16405698/37252442-e2af6ffe-2518-11e8-82a4-881b314392a6.PNG)

It also checks `biHeight`, `biWidth`, `biPlanes` and `biSize`. From the checks we can calculate that `biHeight` is 0x49 and `biWidth` is `0x19c`, according to `MSDN`, `biPlanes`'s value must be set to 1, `biSize` is the size of `BITMAPINFOHEADER` so it's 0x28

![9](https://user-images.githubusercontent.com/16405698/37252443-e2e90dcc-2518-11e8-9521-e6592a386f62.PNG)

Now we know what values it expects from the headers of `ohmygod.bmp` file.

After that it sets the pointer to `0x36` (which is the sum of headers size) from the beginning of the file, it reads 4 bytes and writes the sum of the the first 3 bytes to the buffer, repeating this proces 9 times, for example, if the data is `0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1 0x1`, the buffer would be `0x3 0x3 0x3 0x3 0x3 0x3 0x3 0x3 0x3`

![11](https://user-images.githubusercontent.com/16405698/37252445-e33308a0-2518-11e8-9b0a-df3e97dae0d7.PNG)

Same happens with the next 20 bytes:

![12](https://user-images.githubusercontent.com/16405698/37252446-e356f04e-2518-11e8-874f-2537b3873d61.PNG)

It decreases the first five bytes of the first buffer:

![13](https://user-images.githubusercontent.com/16405698/37252447-e381af28-2518-11e8-9ecc-468d952817e3.PNG)

...and compares it to the second one:

![14](https://user-images.githubusercontent.com/16405698/37252448-e3a7ff98-2518-11e8-95f3-42082eeaf522.PNG)

After successfully checking all aforementioned fields it tries to open the file:

![15](https://user-images.githubusercontent.com/16405698/37252449-e3c9d2b2-2518-11e8-98cf-8a7cef77f2f4.PNG)

After opening the file it prints a congratulation message:

![16](https://user-images.githubusercontent.com/16405698/37252450-e3ee13fc-2518-11e8-908b-99d2df5d8c95.PNG)

The `MessageBox` uses the first 9 bytes as your name, so you can set whichever name you want, but you should adjust the next 5 bytes accordingly to satisfy requests, in case of `_qaz_qaz`:

![17](https://user-images.githubusercontent.com/16405698/37252451-e40eba80-2518-11e8-8658-f2b463c23a19.PNG)

Almost done, what we need to do now is to download some valid `bmp` file from the [Internet](https://www.ece.rice.edu/~wakin/images/lena512.bmp), change header values, change the first 0x3A bytes of data and that's all!

![18](https://user-images.githubusercontent.com/16405698/37252452-e4305528-2518-11e8-8b8f-d2e20644db83.PNG)

`NOTE`: We should remove the `RGBQUAD` structure and append `BITMAPLINE` directly after headers, [`010 Editor`](http://www.sweetscape.com/010editor/)'s `BMP` template is very useful

![19](https://user-images.githubusercontent.com/16405698/37252453-e45330c0-2518-11e8-8d7f-03bcc5e956f3.PNG)

You can download the `crackme` and solution from [here](https://github.com/secrary/sources_from_secrary_posts/blob/master/Imagination.zip)

Any feedback would be greatly appreciated.

Twitter: [@_qaz_qaz](https://twitter.com/_qaz_qaz)