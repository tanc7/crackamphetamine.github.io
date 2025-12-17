---
layout: post
title: "Analyzing Multi‑Stage Detection Evasion Chains in Lab Environments Pt. 2"
date: 2025-12-17
categories: red-teaming cybersecurity
---

## Overview

OK, so this is yet another transcript, and I want to talk about what could be a good use of our hardware breakpoints. The reason why is that in Sector 7 Malware Development Advanced Vol. 1, you can use hardware breakpoints with guard pages—also known as a "no-patch hook."

Using the guard page itself creates a... basically, it's very inaccurate because it can only trigger on a memory page. On average, a memory page is 4,096 bytes. Now, the problem is: what do you want to use a hardware breakpoint on? A guard page and a Vectored Exception Handler (VEH) are not that accurate; they have to hook an entire memory page. Unlike a software breakpoint—which basically drops in int 3 instructions—Sector 7 uses precision hardware breakpoints to move to a specific address or location in memory, just like a software breakpoint.

However, there is a flaw. You could cause exception loops because you don't have granular control; you're trying to break on access to a memory page. So, Renz00h's method of using hardware breakpoints allows you to break on a specific byte, which is the whole point. But the flaw with using hardware breakpoints is that you can only have a maximum of four hardware breakpoints in the modern Intel architecture (for both 32-bit and 64-bit).

So, instead of wasting any amount of time using hardware breakpoints just to redirect to indirect syscalls—which could have been substituted for Beacon Gate—you should reconsider your strategy. Beacon Gate is a method to take the proprietary source code of a Cobalt Strike Beacon (which you don't actually have the source of, you only have the Arsenal Kit) and proxy every call made by that beacon. You want to redirect the most obvious malware behavior—like VirtualAlloc, VirtualProtect, CreateThread, CreateRemoteThread, and their ntdll equivalents—using direct/indirect syscalls or a Hook Chain. A Hook Chain, by the way, is a combination of Hell's Gate, indirect syscalls, and Import Address Table (IAT) hooking through a proxy function.

You don't want to waste your four available functions just on hardware breakpoints to implement Beacon Gate. Also, Beacon Gate is quite loud—although I strongly debate how people call Beacon Gate "loud" these days. The Sector 7 specific course was made maybe over four years ago, right? And a lot of people have pretty much automated this using open-source toolkits that effectively create the template for a Beacon Gate. All you have to do is type down the instruction you want to hook in ntdll.dll, and then you can create a master Import Address Table hooking DLL to use a global hook within the process.

Going back to hardware breakpoint methods: it only works within a thread of the process. Does that make sense? So for your shellcode thread, it would do really bad things; it would constantly assume it can respawn, allocate virtual memory, and respond to scanning attempts. These are not functions you want to be hooking with a hardware breakpoint, guard page, and Vectored Exception Handler combination.

Instead, you should change out those hardware breakpoint targets. Since we only have four slots (Debug Registers 0-3), here is a better allocation:

    Slot 1 (DR0): AmsiScanBuffer

    Slot 2 (DR1): EtwEventWrite (Event Log Tracing for Windows)

    Slot 3 (DR2): NtDelayExecution

        This helps hook sleep calls. Because with sleep obfuscation (like Ekko and variants)—whether they unmap memory pages, add guard pages, unmap and remap memory, or encrypt and decrypt it—the whole point is you don't want EDR to scan. If they scan your memory, it's going to be really weird because they will find strange sections of memory that are not mapped, encrypted, or protected with a guard page. That is really suspicious because when you inject a DLL as shellcode, it’s private memory (unlike LdrLoadDll).

Although some people have gotten it wrong and say not every EDR actually injects their DLLs lately, the majority of EDRs these days still do. If they still do this, that means you can stop EDR from injecting into that malicious process (or host process) simply by spawning a new one from disk and then suspending it; the EDR cannot inject into a suspended process. You could have just dropped all of your direct or indirect syscall bypasses there. That's actually true, by the way. However, as soon as you resume a thread, the EDR will inject that DLL into it.

But Slot 4 (DR3) can still use LdrLoadDll to detect EDR DLLs, and then you can actually hook that reverse DLL. This is going to be hard to do per vendor because there are actually hundreds of EDR vendors. People talk about CrowdStrike and Cylance and all this shit, but they're not the only EDRs in existence. There are Chinese EDRs, Baidu EDRs, Kaspersky (Russian) EDRs... there are actually hundreds. It's really interesting; you should check out the source code of EDRSandblast, where they grab offsets and learn C++ exploits about this.

This is not the "go-to, wreck everything" kind of hardware breakpoint EDR evasion technique, but it will take care of standard scanning. The reason why is that top-tier ones—like Microsoft Defender for Endpoint, up to CrowdStrike, up to Carbon Black—are highly optimized to protect Windows environments. If they can't access what they can't see in Userland (since this is all Userland techniques), they will use their signed driver to access it from the Kernel. They can still walk through the EPROCESS struct in the Kernel if they have your Process ID and observe memory from the Kernel—which you can't detect without your own vulnerable driver exploit.

### Key Notes

- All demonstrations are conducted in isolated lab environments.
- Historical techniques are discussed for defensive research purposes.
- Modern detection systems have mitigations that address these techniques.
