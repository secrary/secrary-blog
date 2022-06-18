---
layout: post
title: "Mastermind crackme by Spider"
---

Download the `crackme` from [`here.`](https://github.com/secrary/sources_from_secrary_posts/files/2089681/mastermind.zip)

The `CrackMe` is created by `Spider` and seems like it's not too easy to solve. We should create a `keygen` and find three hidden easter eggs.
As you will see, my solution is not creating a normal `keygen`, but maybe a more lazy and `hacky` way to solve the `crackme`.

![1](https://user-images.githubusercontent.com/16405698/41216955-e851289c-6d45-11e8-8212-0cb1e9152d6f.PNG)

At `0x0406486` it gets `name` string from a user, calculates `dword` value and writes to `loc_4066B2+1` location,  it overwrites `0xCCCCCCCC`:

![2](https://user-images.githubusercontent.com/16405698/41216956-e8d5f84c-6d45-11e8-8a00-be152005e92e.png)

click [here](https://user-images.githubusercontent.com/16405698/41216956-e8d5f84c-6d45-11e8-8a00-be152005e92e.png) for larger version

![3](https://user-images.githubusercontent.com/16405698/41216957-e90ca680-6d45-11e8-9e67-d7ffc744576a.png)

After getting a `serial` from a user, it checks the serial's length, it must be 26 bytes, and only contain following characters: `0 1 2 3 4 5 6 7 8 9 A B C D E F`, after that it converts the serial into hexadecimal form, for example: `123456789ABCDEFABCDE123456` becomes `0x12 0x34 0x56 0x78 0x9A 0xBC 0xDE 0xFA 0xBC 0xDE 0x12 0x34 0x56`:

![4](https://user-images.githubusercontent.com/16405698/41216958-e941ca18-6d45-11e8-80d6-1d9945fc518c.PNG)

At `0x0406557` it calls `checkOpcode` function, which basically is a huge `switch` statement, the arguments are the `hex` version of the serial and `start_of_some_DISASM_struct` structure.

![5](https://user-images.githubusercontent.com/16405698/41216959-e96f9880-6d45-11e8-9ab6-d7d450fe8e2e.PNG)

I've spent most of the time on analyzing/guessing what this huge function is, it modifies `start_of_some_DISASM_struct` structure based on values from the serial.

I found that inside `sub_406622` function, it interprets the serial as code and calls it, I thought that `checkOpcode` function is maybe some kind of `assembly instruction parser` / `disassembler`, it gets opcodes, modifies `start_of_some_DISASM_struct` structure and returns some value via `eax` register.

![6](https://user-images.githubusercontent.com/16405698/41216960-e99c1cb6-6d45-11e8-9133-462707da05fa.PNG)

For example, `mov eax, 0x12345678` instruction in `hex` form is `B878563412`, in case of `B8` instruction, the function adds 5 to the serial to move to the next instruction:

![7](https://user-images.githubusercontent.com/16405698/41216961-e9cbe77a-6d45-11e8-942a-ad525f56215a.png)

After that it checks if an opcode is allowed:

![8](https://user-images.githubusercontent.com/16405698/41216963-ea01ae1e-6d45-11e8-946e-9b022e8830b6.PNG)

Following opcodes are allowed in our serial:

![9](https://user-images.githubusercontent.com/16405698/41216942-e53ebf16-6d45-11e8-8489-d5df8050d0ec.PNG)

It checks if the number of operands is more than zero and checks if operands are `epb` or `esp` (also there is check for `lea` instruction, etc.), if so it goes to `bad_boy` message.

![10](https://user-images.githubusercontent.com/16405698/41216943-e5682860-6d45-11e8-94d5-9f1d538fa49e.PNG)

![11](https://user-images.githubusercontent.com/16405698/41216944-e59e58ae-6d45-11e8-837a-c2ac8e0afd13.PNG)

After that it calls `masterMind_mainCheck_406622` function, this is where serial checks happen:

![12](https://user-images.githubusercontent.com/16405698/41216945-e6651296-6d45-11e8-8ec3-93d79cb094de.PNG)

`0xCCCCCCCC` will be overwritten by a value derived from a name:

![13](https://user-images.githubusercontent.com/16405698/41216946-e6905708-6d45-11e8-949e-4fd80d06a874.PNG)

`masterMind_mainCheck_406622` is where checks happen and at first glance, seems like its not too easy. It calls a user controlled `serial` as a function, so I tried to hijack execution and jump into `good_boy` message, but that was not too easy as well, because many useful instructions like `push`, `pop`, `mov ebp, ...`, `mov [esp], ...` and etc. are not allowed, length must be equal or less than 13 bytes.

But we still can find useful instructions, I've changed `return` value (`[esp]`) to point `good_boy` message and `ebp` to valid `window` handle:

![untitled](https://user-images.githubusercontent.com/16405698/41226083-ea765060-6d5f-11e8-9670-eb7b4419d112.png)

![vs_asm](https://user-images.githubusercontent.com/16405698/41216951-e7f6b2a4-6d45-11e8-809d-e29a1b214a26.PNG)

Now we have the universal key: `8B442408958B0424047E870424` and `"keygen"` if you wish :)

![14](https://user-images.githubusercontent.com/16405698/41216948-e73f34b2-6d45-11e8-9e1c-c1b02eb62c14.PNG)

![universal_key](https://user-images.githubusercontent.com/16405698/41217229-cc485a52-6d46-11e8-9ce3-92574c5d85fc.gif)

# EASTER EGG #1

If on the main window a user presses any button, execution jumps to `0x04060AD` location

![1](https://user-images.githubusercontent.com/16405698/41217852-b5653fd8-6d48-11e8-8152-a1266e549544.PNG)

We control `0x12345678` value, if we press `a` and `b` it becomes `0x56784142` (`a` == 41, `b` == 42)

![2](https://user-images.githubusercontent.com/16405698/41217853-b5902432-6d48-11e8-85cf-66a6ad2468aa.PNG)

To jump the `0x04060DE` block we need to solve the simple equation:

![3](https://user-images.githubusercontent.com/16405698/41217854-b5b66174-6d48-11e8-8080-6cc5bbc9c44d.PNG)

We need to type `HOLE` while focused on the main window:

![hole](https://user-images.githubusercontent.com/16405698/41217784-7f587c0c-6d48-11e8-9953-3bff18c0bc67.gif)

# EASTER EGG #2

The second easter egg is inside `about window`'s dialog box procedure.

If we click the right mouse button two times in the `about window` area, it confines the cursor to the `about window` (to release the cursor we should right click the mouse button two times again)

![1](https://user-images.githubusercontent.com/16405698/41224409-f432bf98-6d5b-11e8-9815-6bce5ae32435.PNG)

![easter_egg2](https://user-images.githubusercontent.com/16405698/41224361-d0de553e-6d5b-11e8-9d76-034e4dc633c6.gif)

# EASTER EGG #3

If a name is `einstein` it changes the cursor shape to `Einstein`'s image.

![easter_egg_3](https://user-images.githubusercontent.com/16405698/41216950-e7bc97e0-6d45-11e8-9e44-287f754c5040.png)

![easter_egg3](https://user-images.githubusercontent.com/16405698/41224688-996d47bc-6d5c-11e8-8243-b955ff5cd823.gif)

Thank you for your time.

Any feedback would be greatly appreciated.

Discuss on [Reddit](https://www.reddit.com/r/ReverseEngineering/comments/8q9kvj/mastermind_crackme_by_spider/)

Twitter: [@_qaz_qaz](https://twitter.com/_qaz_qaz)