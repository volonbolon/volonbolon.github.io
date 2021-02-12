---
layout: post
title: "System Integrity Protection"
date: 2021-02-12 15:08:10 GMT
tags: Xcode Debug LLDB
description: "Configure SIP to attach LLDB to protected processes"
---

Since 2015, with the introduction of El Capitan, Apple secured the kernel of Mac OS with a new system that walls off certain features from access. Even for root-level accounts. Actually, it was marketed as a **Rootless** system. 

The additional isolation of system components from accounts with root privileges helps to prevent *malware* from being able to gain access to the system, where it could embed itself and take advantage of all of the system services running on a Mac.

The System Integrity Protection hardened the Mac by preventing modifications to the following locations: `/System`, `/usr`, `/bin`, `/sbin`, and all apps preinstalled by Apple. Apple reserved for itself the privilege to write to system files by signing apps and process with special entitlements. Only Apple-signed system processes can write to system locations. 

So far, so good, unless the process is signed by Apple, it cannot write to specific locations. But SIP goes even further, by preventing processes to be attached to other processes. This prevents code injection or runtime attachment to system processes, techniques used by malware to force privileged processes to run the malware code.

With Mojave, SIP was extended to third-party apps. This should protect them from being tampered with, having code injected, or having processes attached to them.

Although Rootless is a substantial leap forward in security, it introduces some annoyances as it makes programs harder to debug. Specifically, it prevents other processes from attaching a debugger to programs Apple signs. For instance, if we try to attach lldb to Finder we will get an error: 

```
❯ lldb -n Finder
(lldb) process attach --name "Finder"
error: attach failed: attach failed (Not allowed to attach to process.  Look in the console messages (Console.app), near the debugserver entries when the attached failed.  The subsystem that denied the attach permission will likely have logged an informative message about why it was denied.)
(lldb)
```

### Toggling SIP
Although Apple would like you to always keep SIP turned on, it can be disabled. If you choose to do that, it is important to turn it on as soon as possible. Or even better, set up a Virtual Machine using VMWare and disable SIP only on the VM. 

To check the current status of SIP, we need to run: 

```
❯ csrutil status
System Integrity Protection status: enabled.
```

SIP can’t be toggled directly from within the currently running version of the Mac OS; instead, the Recovery volume is used to add a boot argument to your Mac’s NVRAM that controls the SIP system.

To disable SIP, perform the following steps:

1. Restart your macOS machine.
2. When the screen turns blank, hold down Command + R until the Apple boot logo appears. This will put your computer into Recovery Mode.
3. Find the Utilities menu from the top and then select Terminal.
4. Enter the command `csrutil disable && reboot`

Because SIP is controlled through Mac’s NVRAM, enabling or disabling SIP affects all versions of the Mac operating system you’ve installed. If you disable SIP to allow an app to be installed in OS X El Capitan, SIP will also be disabled if you should boot into macOS Mojave that you installed on another volume. SIP is a global setting that affects all systems installed on your Mac.

With SIP disable, you can attach lldb to any process: 

```
❯ csrutil status
System Integrity Protection status: disabled.
❯ lldb -n Finder
(lldb) process attach --name "Finder"
Process 390 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = signal SIGSTOP
    frame #0: 0x00007fff20327e7e libsystem_kernel.dylib` mach_msg_trap  + 10
libsystem_kernel.dylib`mach_msg_trap:
->  0x7fff20327e7e <+10>: ret
    0x7fff20327e7f <+11>: nop
libsystem_kernel.dylib'mach_msg_overwrite_trap:    0x7fff20327e80 <+0>: mov    r10, rcx
    0x7fff20327e83 <+3>:  mov    eax, 0x1000020
    0x7fff20327e88 <+8>:  syscall
    0x7fff20327e8a <+10>: ret
    0x7fff20327e8b <+11>: nop
libsystem_kernel.dylib'semaphore_signal_trap:    0x7fff20327e8c <+0>: mov    r10, rcx
Target 0: (Finder) stopped.

Executable module set to "/System/Library/CoreServices/Finder.app/Contents/MacOS/Finder".
Architecture set to: x86_64h-apple-macosx-.
```