---
layout: post
title: "Signaling for large scale adversarial operations"
date: 2025-12-18
categories: red-teaming cybersecurity
---

## Overview

Okay, so let's talk about signaling, because for solo operators or small teams, you need a lot of automation. You might be having a lot of questions. For example, how do you know to trigger a ransomware attack after you stole the files? Right? From one machine to a machine under a domain, to the machines between multiple domain controllers under an entire forest of an enterprise.

So, there are multiple "on-disk" and quote-unquote "in-memory" techniques. It doesn't really qualify as 100% in-memory, but we'll go through it. So there are two ways on modern Windows machines that you can hide data, okay? You can hide data using the command line, cmd.exe; you can do it with PowerShell. You can create hidden folders in directories, which is an on-disk technique, or you can do one of two things.

The first, most common method is an Alternate Data Stream (ADS). You can put an alternate data stream in any file on a modern NTFS file system. So you can add a little JSON thing, a configuration, to control when you are stealing files and when you are activating your ransomware attack, right? You can do things like add counters, send it back to your automated C2, which then the logic will execute a ransomware attack. And this helps streamline operations.

So let's talk about Alternate Data Streams. Alternate Data Streams, very common thing, but there is a catch. One: it only works on NTFS file systems. And two: you cannot transfer a file with a written alternate data stream and preserve that alternate data stream. So if you found, like, a Jenkins install, and it has obviously a Readme.txt documentation file, if you grab, like, let's say, activate_ransom=off and wrote it as an Alternate Data Stream into that Readme documentation, and then you grabbed it, you won't see that alternate data stream when it's transferred remotely back to your machine, or exfiltrated. So that's another catch. It has to remain resident on the disk of the target.

And the other method is Extended Attributes, which is another way that can also work on both FAT32 and NTFS file systems. Both of these means do affect the disk, though, by the way—for NTFS Alternate Data Streams, and for both FAT32 and NTFS Extended Attributes.

And then there's a third way. The third way is you can modify the Registry, and this is where the thing about the debates come in. So the Registry supposedly is not on the disk, but the settings are loaded into memory on every reboot. You can clone the Registry hive; when you start up Windows, a file handle is open to the Registry, so you cannot copy the current Registry. You have to make a backup of the Registry hive. So when you steal the Registry hive to crack the passwords—you know, common really old-school techniques back in like the year 2014—well, that's actually an outdated backup of the Registry hive that is backed up on reboot. But so there's a lot of debate on whether or not the Registry is really in-memory or it's on the disk.

But you can query the Registry key. But the good thing about the Registry hive is that non-administrative users can write to their own Registry entry. And it's really easy to hide an on-and-off switch for signaling in the Registry because there are so many registry settings. You can make one up for Internet Explorer, you know? Like let's make one up right now: "Enable SSL/TLS logging." You know, because there's actually tricks that you can do to your web browser on Firefox—what else? Brave, Chrome—and you can actually preserve SSL/TLS connections to decrypt your own streams, and you can do that, right? But when it doesn't really do that, it's just another way of saying we just have a fake word, and it stores a DWORD: zero for off, one for on. And we can read that as the current user, which is not an administrator.

So that was the beauty of the Registry, because everything Windows since like the year of 2000, the NT Kernel, Windows 2000, you can store registry keys. And malware does use that to store registry keys to do signaling. And for that reason, maybe the Registry is the best way to do this, because you can just make up any of them. However, they are commonly cataloged as, unfortunately, Indicators of Compromise (IoCs), so you should think twice. Instead, just use Alternate Data Streams or Extended Attributes instead, among other ways to hide data.

So think about that, because what if you were able to enumerate the entire domain automatically, you transfer a lot of files—and I'm not really going to go through much of file transfers outside of saying, hey, it's really trivial to write an in-memory file stealer. It's really trivial to steal credentials in a browser, by the way, using a Data Protection API (DPAPI), which is way more important in info stealers. But for transferring files, you could have just made a bunch of S3 accounts and then dropped Rclone, and then waited for Rclone to exit, and then change the registry key to ransom the machine, or the domain, to "True" or "1". You read that key to "True," and then you start the ransomware attack.


### Key Notes

- All demonstrations are conducted in isolated lab environments.
- Historical techniques are discussed for defensive research purposes.
- Modern detection systems have mitigations that address these techniques.
