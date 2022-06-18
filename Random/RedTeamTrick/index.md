---
layout: post
title: "Simple Trick For Red Teams"
---

If you have an unsigned binary, which requires `administrator` privileges, when a target runs the binary following window will show up:

![unsigned_binary](https://user-images.githubusercontent.com/16405698/66422419-57829f80-ea12-11e9-8bf1-968c4e3ed985.png)

The window's header is yellow, which means the binary is not signed, also in the current example, a publisher is unknown.

There is a way to request `administrator` privileges a bit more convincing way, execute `cmd.exe` with elevated privileges and run your binary from the `cmd.exe` process.

Code:
<script src="https://gist.github.com/secrary/b87908ddbeb8ee52f99ae5ec490f6792.js"></script>

With this approach, the window is blue (binary is signed) and also publisher is `Microsoft`, it's more likely that the target will approve the request:

![signed_cmd](https://user-images.githubusercontent.com/16405698/66422580-9a447780-ea12-11e9-8a26-1bdc1a92f19a.gif)


Twitter: [@_qaz_qaz](https://twitter.com/_qaz_qaz)