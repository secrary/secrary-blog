---
layout: post
title: "Debugging Windows Services For Malware Analysis / Reverse Engineering"
---

`A service, also known as a Windows service, is a user-mode process designed to be started by Windows without human interaction. It is started automatically at system boot, or by an application that uses the service functions included in the Win32 API. A service can also be started by a human user through the Services control panel utility. Every service must conform to the interface rules of the service control manager (SCM).` - [MSDN](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-a-service-application)

During pre-`Windows Vista` era services were ordinary processes on the same session. After `Windows Vista` and later everything else, except services,  was moved out of `Session 0`. In general, executables from `Session 0` don't communicate with our desktop session - `Session 1`. This makes it hard to debug them.

Unlike a normal application, we cannot start a service by entering its name on the command line, or by calling `CreateProcess`. Instead, we need to rely on the `Service Control Manager` to start and stop the process.

`NOTE: I assume we don't have access to the source code of the service application`

When the `service control manager` starts a service process, it waits for the process to call the `StartServiceCtrlDispatcher` function, if we run a service executable from any other context we will get `ERROR_FAILED_SERVICE_CONTROLLER_CONNECT` `(0x427)`

![1](https://user-images.githubusercontent.com/16405698/39088572-056a5f58-45a4-11e8-8752-861942246bd4.png)

* How can we debug a service?

We can start a service and attach a debugger such as [`WinDbg`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools):

![2](https://user-images.githubusercontent.com/16405698/39088573-058f998a-45a4-11e8-920d-ac351e006496.png)

* What if we need to a debug the beginning of a process execution?

To run a debugger before a process execution we need to create a key under `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\ProgramName` registry key or use [`GFlags (the Global Flags Editor)`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/gflags) and set a debugger.
We cannot use [`WinDbg`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools) directly due to a service and `WinDbg` execute at `session 0` and we cannot see [`WinDbg`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools) UI from `session 1`, but we can use [`NTSD`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-using-cdb-and-ntsd) as the server debugger on `session 0` and [`WinDbg`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools) on `session 1` as the client debugger.

![3](https://user-images.githubusercontent.com/16405698/39088575-05b5b6c4-45a4-11e8-8d38-46799d294ff9.PNG)

[`NTSD`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-using-cdb-and-ntsd) with the `-noio` option causes [`NTSD`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-using-cdb-and-ntsd) to run without any console of its own, accessible only through the remote connection.

There is an important registry value under `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control` key: `ServicesPipeTimeout`

`The significance of this value is that a clock starts to run when each service is launched, and when the timeout value is reached, any debugger attached to the service is terminated. Therefore, the value you choose should be longer than the total amount of time that elapses between the launching of the service and the completion of your debugging session... For example, a value of 60,000 is one minute, while a value of 86,400,000 is 24 hours. When this registry value is not set, the default timeout is about thirty seconds...` - [MSDN](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/preparing-to-debug-the-service-application)

We need to reboot the system after changing the value.

![4](https://user-images.githubusercontent.com/16405698/39088576-05d8def6-45a4-11e8-8102-b1c40a55840d.PNG)

After that, run [`WinDbg`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools) with the following commands: `.\windbg.exe -remote tcp:server=localhost,port=55555`.

We can add a breakpoint at the entry point of a sample by adding `AddressOfEntryPoint` to start the address of the loaded executable:

![ep](https://user-images.githubusercontent.com/16405698/39088571-05447c66-45a4-11e8-9fff-d05307fa3231.PNG)

That's all, we are at the entry point.

Any feedback would be greatly appreciated. [@_qaz_qaz](https://twitter.com/_qaz_qaz)

Discuss on [Reddit](https://www.reddit.com/r/ReverseEngineering/comments/8dywlm/debugging_windows_services_for_malware_analysis/)