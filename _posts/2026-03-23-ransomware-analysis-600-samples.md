---
title: "I Analyzed 600+ Real Ransomware Samples from the Wild — What They Do Beyond Just Encrypting Files"
date: 2026-03-23 09:00:00 +0000
categories: [cybersecurity, malware-analysis, threat-research]
tags: [ransomware, reverse-engineering, edr, endpoint-protection, malware]
pin: true
toc: true
comments: true
math: false
mermaid: false
img_path: /assets/img/posts/2026-03-23-ransomware/
---

In early 2026 I spent roughly one full month deeply analyzing over 600 live ransomware samples collected from more than 100 different families.
The goal of the project was straightforward but aggressive: execute real-world ransomware in controlled environments while closely monitoring and stress-testing a commercial endpoint protection / EDR product.
Quick Dataset Overview

Total samples: 600+
Families covered: 100+ (average ~10 samples per major family)
Collection period: ~1 month
Main sources:
ANY.RUN threat intelligence feeds
MalwareBazaar
VX-Underground archive
Public GitHub repositories
Limited private intel drops

Notable families (partial list):
LockBit (3.0 & 4.0 variants)
Akira
Cl0p
REvil / Sodinokibi
Conti
Ryuk
BlackBasta
BlackMatter
DarkSide
Maze
Hive
Medusa
BianLian
Qilin
Rhysida
Avaddon
Nefilim
Pysa / Mespinoza
HelloKitty (Vice Society overlap)
Trigona
RA World (formerly RA Group)
BlackCat / ALPHV
Ragnar Locker
…and many more


All detonations happened in isolated lab environments (physical hardware was mandatory after early cloud/VM detection failures).
Key Finding #1 – They Know When You Are Analyzing Them
A non-trivial number of families embed strong anti-analysis and environment-awareness checks.
Most aggressive VM detectors:

BlackCat (ALPHV)
Ragnar Locker

Typical checks observed:

Registry artifacts (HKLM\SOFTWARE\Oracle\VirtualBox Guest Additions, etc.)
Running processes (VBoxService.exe, VBoxTray.exe, etc.)
MAC address OUI ranges
CPUID instruction results
Filesystem / driver fingerprints

Cloud detection:

~4 samples specifically checked for AWS / Azure / GCP
Methods: instance metadata service (http://169.254.169.254/), cloud-specific drivers, hostname patterns
Behavior on detection: complete refusal to proceed — no encryption, no ransom note, no disk writes, silent exit

“When they spotted it they didn't execute and I had to do some analysis… I had to run in the physical desktop environment.”
Lesson for analysts: cloud-hosted sandboxes and default VM configurations are increasingly blind spots.
Key Finding #2 – Spyware & Credential Theft Happens First
Modern ransomware almost always contains a full stealer component that runs before or in parallel with encryption.
Common stolen items:

SAM / SYSTEM / SECURITY registry hives → local account password hashes
Browser credentials (Chrome, Edge, Firefox via DPAPI or direct SQLite access)
Desktop screenshots (including current wallpaper — likely for victim identification)
Real-time keylogger
Clipboard content (crypto addresses, etc.)

Exfiltration methods observed:

Telegram bots — by far the most common and easiest to implement
Custom C2 HTTP/HTTPS POSTs
Occasionally Discord webhooks

Exfil often occurs quietly and can be delayed until user activity is detected (mouse/keyboard events), reducing sandbox hits.
This enables classic double extortion (encrypt + data leak threat) or even triple extortion (DDoS threat on top).
Key Finding #3 – Screen Takeover & Psychological Warfare
Several families go far beyond file locking — they actively terrorize the user.
Observed behaviors:

Full-screen black overlay displaying Bitcoin / Monero wallet address
Playback of looping meme videos or corrupt hard disk grinding sounds (the classic failing HDD noise)
Complete desktop freeze: Explorer.exe killed or hooked, taskbar/input disabled
Interference with File Explorer and Notepad (prevent opening or viewing files)
Wallpaper replacement (sometimes with taunting images/text)

Families most aggressive here:

Medusa
RA World

One particularly nasty Medusa sample abused bcdedit to:

Disable Safe Mode (bcdedit /set {default} safeboot Minimal → later removed)
Disable automatic recovery
Delete shadow copies (vssadmin delete shadows /all /quiet)

Result: system became effectively unrecoverable without full OS reinstall.
I personally had to wipe and reinstall the test machine from scratch.
“It showed a black screen with wallet address… showed a video type… looping video with the corrupt disk sound… effected the boot loader and I had to install a new OS.”
This crosses from malware into psychological operations.
Key Finding #4 – Dropper & Anti-Forensic Techniques
Multiple families used multistage / self-preserving behaviors:

Self-replacing executable:
Drop new binary in temp / appdata / programdata
Delete original file
Execute the new copy
(RA World and similar multistage families were frequent offenders)

Creation of multiple identical copies across different directories
Repeated environment (VM/cloud) checks even after initial execution
Registry / service persistence for reboot survival
Wallpaper changes, desktop icon manipulation

These techniques significantly raise the bar for both AV/EDR detection and incident response cleanup.
The Single Most Important Lesson
After touching 600+ samples, the reality is clear:
“Ransomware doesn’t only encrypt files — it corrupts some boot loader, makes copies of the file, communicates with the C2 and also implant keystrokes, changes wallpaper, steals creds, and many other [behaviors] as mentioned earlier.”
Encryption is now the last visible stage of a much longer, multi-purpose infection chain.
Most detection strategies still over-focus on file-encryption patterns and ransom notes.
The real compromise happens in the stealthy prelude: credential theft, C2 communication, boot configuration sabotage, screen control.
Coming in Part 2
I will publish the second part soon:
How I built my own custom ransomware PoC and which specific evasion techniques successfully bypassed multiple commercial EDR / AV / XDR products during testing.
Stay tuned.

Questions for readers
Feel free to comment below:

Which ransomware family currently worries you the most?
Have you ever seen ransomware play meme videos or fake disk failure sounds during infection?
Blue-team perspective: what’s the single detection rule / behavior you wish vendors prioritized more (bcdedit abuse, Telegram exfil, self-replacing binaries, etc.)?

Thanks for reading — stay safe out there.