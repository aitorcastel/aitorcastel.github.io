---
layout: post
title:  "Abusing SFX Files - Part 1"
date:   2025-04-13 12:25:00 +0100
categories: malware
---

> This post is an old backup of the same article I wrote in 2023. It’s just a copy-paste, but I hope you still find it useful.

## Introduction

The other day, I saw a post by [Jai Minton][jai-linkedin] on the [CrowdStrike blog][crowdstrike-blog], where he discussed how the CrowdStrike team discovered an intrusion that started with the use of a seemingly empty SFX archive.

After reading the post (which I highly recommend), I became curious about how it worked and wanted to replicate it. This post is based on how to perform an attack of this type.

I will soon publish other related entries on how to improve the attack. For now, let’s start with something simple.

## Creating a PoC

For this first tutorial, I used only a Windows 10 VM (with RDP enabled) and WinRAR.

The idea is to backdoor a machine where we have admin privileges and an exposed RDP, following the attack chain that Jai described in his post.

### Steps

We need WinRAR installed on the machine. Create an empty file, right-click on it, and select the option `Add to archive…`.

**Note:** The file name will appear when we launch our SFX, so for better operational security, consider giving it a name such as “Windows Update” or something similar.

After that:

1. In the WinRAR GUI, go to **General > Archiving options > Create SFX archive** (you’ll notice the file extension changes to `.exe`).
2. Set a password.
3. Go to the **Advanced > SFX Options** tab.

#### Configuration:

3.1 **Setup > Run after extraction**: Add the following commands in the dialog box:

{% highlight txt %}
powershell.exe
cmd.exe
taskmgr.exe
{% endhighlight %}

3.2 **Modes > Silent Mode > Hide all**

3.3 **Update > Update Mode > Extract and replace files**

3.4 **Overwrite mode > Skip existing files**

Press **OK** and create the file. Check that everything works correctly.

Now, we need to create “persistence” using `utilman.exe` with the debugger option linked to our SFX file. For that, we need to save the SFX file on the Windows 10 victim machine and note its path.

> **Note:** For this PoC, disable Windows Defender. The “Debugger” option in the registry is detected by Defender.

Use the following command to automate the process (thanks to Jai):

{% highlight powershell %}
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\utilman.exe" /v "debugger" /d "[REDACTED PathToSFXArchive]" /f
{% endhighlight %}

Now, we can close our session and try to connect to the machine via RDP. When we connect, we just click on the “Ease of Access” options, and our SFX will be launched. Enter the password, and now you have an `NT AUTHORITY\SYSTEM` session.

## Demo

You can find a video demonstrating this attack method here: [PoC][video-poc].

Thanks! See you next time in another post.

[jai-linkedin]: https://www.linkedin.com/in/jaiminton/
[crowdstrike-blog]: https://www.crowdstrike.com/en-us/blog/self-extracting-archives-decoy-files-and-their-hidden-payloads/
[video-poc]: https://www.youtube.com/watch?v=nR6r0pI57ks